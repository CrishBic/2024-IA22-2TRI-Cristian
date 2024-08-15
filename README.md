## Iniciando um projeto Node.js com TypeScript

Crie um diretório para o projeto e acesse-o pelo vscode, abra o terminal e siga os passos abaixo.

```bash
npm init -y
npm install express cors sqlite3 sqlite
npm install --save-dev typescript nodemon ts-node @types/express @types/cors
npx tsc --init
mkdir src
touch src/app.ts
```

## Configuranado o `tsconfig.json`

Mude na linha ```"outDir": "./",``` para ```"outDir": "./dist",```

**Dica:** para encontrar a linha com facilidade utiliza o ```ctrl+f5``` e digite a linha que você gostaria de encontrar.

Seu arquivo de configuração do compilador do TypeScript ficará + ou - assim:

```json
{
    "compilerOptions": {
        "module": "commonjs",                                
        // "outDir": "./dist",                                 
    }
}
```

 Mude na linha ```"rootDir": "./",``` para ```"rootDir": "./src",```
 
Seu arquivo de configuração do compilador do TypeScript ficará + ou - assim:

```json
{
    "compilerOptions": {
        "module": "commonjs",                                
        // "rootDir": "./src",                                  
    }
}
```

## Configurando o `package.json`

Edite o seguinte script ao seu `package.json` na aba `"scripts":` 
e adicionando `"dev": "npx nodemon src/app.ts"`.

Quando terminar ficará assim:

```json
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
  "dev": "npx nodemon src/app.ts"
}
```

**Obs:** não esqueça de adicionar uma `,` na linha do `"test"`.

## Criando arquivo inicial do servidor

Na pasta `src` vá até o arquivo `app.ts` criado anteriormente e adicione o seguinte código:

```typescript
import express from 'express';
import cors from 'cors';

const port = 3333;
const app = express();

app.use(cors());
app.use(express.json());

app.get('/', (req, res) => {
  res.send('Hello World');
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```

## Inicializando o servidor

Para iniciar, você  deverá instalar a biblioteca **REST Client**, para fazer isso clique no icone de quatro quadrados que esta sendo desmontado e pesquise `REST client` e clique em instalar. Aguarde a instalaçao para seguir o proximo passo.

Agora para ver sua pagina, digite o seguinte comando no Terminal.

```node
npm run dev
```

Se tudo ocorrer bem, você verá a mensagem `Server running on port 3333` no terminal.

## Testando o servidor

Abra o navegador e acesse `http://localhost:3333`, você verá a mensagem `Hello World`.

## Configurando o banco de dados

Crie um arquivo `database.ts` dentro da pasta `src` e adicione o seguinte código.

```typescript
import { open, Database } from 'sqlite';
import sqlite3 from 'sqlite3';

let instance: Database | null = null;

export async function connect() {
  if (instance) return instance;

  const db = await open({
     filename: './src/database.sqlite',
     driver: sqlite3.Database
   });
  
  await db.exec(`
    CREATE TABLE IF NOT EXISTS users (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      name TEXT,
      email TEXT
    )
  `);

  instance = db;
  return db;
}
```

## Adicionando o banco de dados ao servidor

Vá à pasta `app.ts` e troque todo o código pelo código seguinte:

```typescript
import express from 'express';
import cors from 'cors';
import { connect } from './database';

const port = 3333;
const app = express();

app.use(cors());
app.use(express.json());

app.get('/', (req, res) => {
  res.send('Hello World');
});

app.post('/users', async (req, res) => {
  const db = await connect();
  const { name, email } = req.body;

  const result = await db.run('INSERT INTO users (name, email) VALUES (?, ?)', [name, email]);
  const user = await db.get('SELECT * FROM users WHERE id = ?', [result.lastID]);

  res.json(user);
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```

# Testando a inserção de dados

Crie um arquivo chamada `teste.http` **fora da pasta src** e adicione o seguinte codigo

```http
POST (https://localhost:3333) HTTP/1.1
Content-Type: application/json
Authorization: token xxx

{
  "name": "John Doe",
  "email": "john@example.com"
}

````

## Listando os usuários

Adicione a rota `/users` ao servidor.

```typescript
app.get('/users', async (req, res) => {
  const db = await connect();
  const users = await db.all('SELECT * FROM users');

  res.json(users);
});
```

## Editando um usuário

Adicione a rota `/users/:id` ao servidor.

```typescript
app.put('/users/:id', async (req, res) => {
  const db = await connect();
  const { name, email } = req.body;
  const { id } = req.params;

  await db.run('UPDATE users SET name = ?, email = ? WHERE id = ?', [name, email, id]);
  const user = await db.get('SELECT * FROM users WHERE id = ?', [id]);

  res.json(user);
});
```

## Deletando um usuário

Adicione a rota `/users/:id` ao servidor.

```typescript
app.delete('/users/:id', async (req, res) => {
  const db = await connect();
  const { id } = req.params;

  await db.run('DELETE FROM users WHERE id = ?', [id]);

  res.json({ message: 'User deleted' });
});
```