# Iniciando um projeto Node.js com TypeScript

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

**Dica:** para encontrar a linha com facilidade utiliza o ```ctrl+f5``` e digite qual a linha que você gostaria de encontrar.

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

app.get('/users', async (req, res) => {
  const db = await connect()
  const users = await db.all('SELECT * FROM users')
  res.json(users)
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

Vá à pasta `app.ts` e troque **TODO** o código pelo código seguinte:

```typescript
import express from 'express';
import cors from 'cors';
import { connect } from './database';

const port = 3333;
const app = express();

app.use(cors());
app.use(express.json());

app.get('/users', async (req, res) => {
  const db = await connect()
  const users = await db.all('SELECT * FROM users')
  res.json(users)
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

Crie um arquivo chamada `teste.http` **dentro da pasta src** e adicione o seguinte código

```http
POST https://localhost:3333 HTTP/1.1
Content-Type: application/json
Authorization: token xxx

{
  "name": "John Doe",
  "email": "john@example.com"
}

````

## Listando os usuários

### Adicione a rota `/users` ao servidor.

No arquivo `app.ts` adicione o seguinte codigo abaixo da linha `10` com o comando de `app.use(express.static(__dirname + '/../public'))`

```typescript
app.get('/users', async (req, res) => {
  const db = await connect();
  const users = await db.all('SELECT * FROM users');

  res.json(users);
});
```

## Editando um usuário

### Adicione a rota `/users/:id` ao servidor.

No mesmo arquivo adicione o seguinte código abaixo do `app.post`

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

### Adicione a rota `/users/:id` ao servidor.

No mesmo arquivo adicione o seguinte código abaixo do `app.post`

```typescript
app.delete('/users/:id', async (req, res) => {
  const db = await connect();
  const { id } = req.params;

  await db.run('DELETE FROM users WHERE id = ?', [id]);

  res.json({ message: 'User deleted' });
});
```

## Parte do Cadastro com index.html

Agora crie uma pasta com o nome `public` e um arquivo `index.html`

Após criar o arquivo, adicione o seguinte código no arquivo:

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>

<body>
  <form>
    <input type="text" name="name" placeholder="Nome">
    <input type="email" name="email" placeholder="Email">
    <button type="submit">Cadastrar</button>
  </form>

  <table>
    <thead>
      <tr>
        <th>Id</th>
        <th>Name</th>
        <th>Email</th>
        <th>Ações</th>
      </tr>
    </thead>
    <tbody>
      <!--  -->
    </tbody>
  </table>

  <script>
    // 
    const form = document.querySelector('form')

    form.addEventListener('submit', async (event) => {
      event.preventDefault()

      const name = form.name.value
      const email = form.email.value

      await fetch('/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ name, email })
      })

      form.reset()
      fetchData()
    })

    // 
    const tbody = document.querySelector('tbody')

    async function fetchData() {
      const resp = await fetch('/users')
      const data = await resp.json()

      tbody.innerHTML = ''

      data.forEach(user => {
        const tr = document.createElement('tr')
        tr.innerHTML = `
          <td>${user.id}</td>
          <td>${user.name}</td>
          <td>${user.email}</td>
          <td>
            <button class="excluir">excluir</button>
            <button class="editar">editar</button>
          </td>
        `

        const btExcluir = tr.querySelector('button.excluir')
        const btEditar = tr.querySelector('button.editar')

        btExcluir.addEventListener('click', async () => {
          await fetch(`/users/${user.id}`, { method: 'DELETE' })
          tr.remove()
        })

        btEditar.addEventListener('click', async () => {
          const name = prompt('Novo nome:', user.name)
          const email = prompt('Novo email:', user.email)

          await fetch(`/users/${user.id}`, {
            method: 'PUT',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ name, email })
          })

          fetchData()
        })

        tbody.appendChild(tr)
      })
    }

    fetchData()
  </script>
</body>

</html>
```

Pare o processo do terminal com o atalho `ctrl + c` e rode o arquivo novamente com o comando no terminal `npm run dev` para rodar a tabela de usuários cadastrados no seu site.