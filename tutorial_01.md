# Tutorial: Criando um App de Lista de Tarefas com Next.js

Neste tutorial, você aprenderá como criar um Web App de Lista de Tarefas usando Next.js com TypeScript e Tailwind CSS. O objetivo é criar uma lista de tarefas simples, com funcionalidades para adicionar e remover tarefas. Futuramente, você poderá integrar um banco de dados e fazer deploy, mas essa parte será abordada em outra etapa.

## 1. Configuração do Projeto
### Passo 1: Criar o Projeto Next.js com TypeScript
No terminal CMD do Windows, navegue até a pasta (diretório) onde você salva seus projetos e execute o seguinte comando para criar o projeto:

```bash 
npx create-next-app@latest to-do-app
```

```bash
√ Would you like to use TypeScript? ... No / Yes
√ Would you like to use ESLint? ... No / Yes
√ Would you like to use Tailwind CSS? ... No / Yes
√ Would you like to use `src/` directory? ... No / Yes
√ Would you like to use App Router? (recommended) ... No / Yes
√ Would you like to customize the default import alias (@/*)? ... No / Yes
```

Esse comando vai criar uma pasta (diretório) com o nome que você criou seu projeto, no caso *to-do-app*

Entre na pasta do projeto com o comando abaixo:

```bash
cd to-do-app
```

### Passo 2: Instalar Tailwind CSS

Instale o Tailwind CSS e suas dependências:

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```
Obs: Você precisa estar na pasta do projeto antes de fazer a instalação acima, pois essa instalação é local no projeto. Para cada projeto que você for utilizar o tailwind é preciso executar os comandos acima.

Abra o projeto no editor de código VSCode:

```bash
code .
```

Configure o arquivo tailwind.config.js com o conteúdo abaixo:

```ts
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    './src/**/*.{js,ts,jsx,tsx}', // Adiciona o tailwind nos arquivos do diretório /src com extesão .js .ts .jsx .tsx
  ],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

Agora, edite o arquivo *./src/app/globals.css* e adicione as seguintes linhas:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Essa configuração importa o tailwind para o CSS global da nossa aplicação.

### Passo 3: Limpar o Projeto
Remova o conteúdo padrão do arquivo *./src/app/page.tsx*, pois vamos começar do zero.

## 2. Construção do App de Lista de Tarefas
### Passo 4: Criar a Estrutura da Página
No arquivo *src/app/page.tsx*, crie a estrutura do nosso aplicativo com o código abaixo:

```tsx
"use client";

import React, { useState } from "react";

type Task = {
  id: number;
  text: string;
  dueDate: string;
};

export default function Home() {
  const [tasks, setTasks] = useState<Task[]>([]);
  const [newTask, setNewTask] = useState("");
  const [dueDate, setDueDate] = useState("");

  const addTask = () => {
    if (!newTask || !dueDate) return;

    const newTaskItem: Task = {
      id: Date.now(),
      text: newTask,
      dueDate: dueDate,
    };

    setTasks([...tasks, newTaskItem]);
    setNewTask("");
    setDueDate("");
  };

  const deleteTask = (id: number) => {
    setTasks(tasks.filter((task) => task.id !== id));
  };

  return (
    <div className="min-h-screen bg-gray-900 flex items-center justify-center">
      <div className="bg-gray-800 p-8 rounded shadow-lg w-full max-w-md">
        <h1 className="text-2xl font-bold mb-4 text-center text-gray-100">Lista de Tarefas</h1>
        <div className="mb-4">
          <input
            type="text"
            placeholder="Descrição da tarefa"
            value={newTask}
            onChange={(e) => setNewTask(e.target.value)}
            className="w-full p-2 mb-2 border border-gray-600 rounded bg-gray-700 text-gray-300"
          />
          <input
            type="datetime-local"
            value={dueDate}
            onChange={(e) => setDueDate(e.target.value)}
            className="w-full p-2 mb-2 border border-gray-600 rounded bg-gray-700 text-gray-300"
          />
          <button
            onClick={addTask}
            className="w-full bg-blue-600 text-white p-2 rounded hover:bg-blue-700 transition"
          >
            Adicionar Tarefa
          </button>
        </div>

        <ul className="space-y-2">
          {tasks.map((task) => (
            <li
              key={task.id}
              className="flex justify-between items-center p-2 bg-gray-700 rounded"
            >
              <div>
                <p className="text-gray-200">{task.text}</p>
                <p className="text-sm text-gray-500">{task.dueDate}</p>
              </div>
              <button
                onClick={() => deleteTask(task.id)}
                className="bg-red-500 text-white p-1 rounded hover:bg-red-600 transition"
              >
                Excluir
              </button>
            </li>
          ))}
        </ul>
      </div>
    </div>
  );
}
```

Explicação do Código:


## 3. Executar o App
### Passo 5: Executar o App
Para rodar o servidor de desenvolvimento, use o comando:

```bash
npm run dev
```

Agora, acesse *http://localhost:3000* no navegador para visualizar seu app de Lista de Tarefas.