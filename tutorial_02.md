# Tutorial: Integrando Banco de Dados ao App de Lista de Tarefas

Neste tutorial, você aprenderá como integrar um banco de dados PostgreSQL ao seu aplicativo de lista de tarefas, utilizando Prisma como ORM e NeonDB como servidor.

## **1. Configurar o Banco de Dados PostgreSQL no NeonDB**

### 1.1. **Criar uma Conta no NeonDB**
   - Acesse [NeonDB](https://neon.tech/) e crie uma conta, se ainda não tiver uma.

### 1.2. **Criar um Novo Projeto**
   - Após o login, clique em "Create Database" para criar um novo banco de dados.
   - Escolha um nome para o banco de dados e clique em "Create".

### 1.3. **Obter a URL de Conexão**
   - Depois de criar o banco de dados, você verá as informações de conexão.
   - Copie a URL de conexão fornecida (deve ser algo como `postgresql://USER:PASSWORD@HOST:PORT/DATABASE`).

## **2. Configurar o Prisma no Projeto**

### 2.1. **Atualizar o Arquivo .env**
   - No seu projeto, localize o arquivo `.env` na raiz.
   - Substitua o valor da variável `DATABASE_URL` pela URL de conexão do NeonDB:
     ```bash
     DATABASE_URL="postgresql://USER:PASSWORD@HOST:PORT/DATABASE"
     ```

### 2.2. **Instalar o Prisma Client**
   - Caso ainda não tenha feito, instale o Prisma Client:
     ```bash
     npm install @prisma/client
     ```

### 2.3. **Configurar o Prisma Client no Next.js**

   - No diretório `src/`, crie uma nova pasta chamada `lib/`.
   - Dentro de `lib/`, crie o arquivo `prisma.ts` com o seguinte conteúdo:

   ```ts
    import { PrismaClient } from '@prisma/client';

    const globalForPrisma = global as unknown as { prisma: PrismaClient };

    export const prisma = globalForPrisma.prisma || new PrismaClient();

    if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;
   ```

## **3. Definir o Modelo de Dados**

### 3.1. **Atualizar o Arquivo schema.prisma**
   - No diretório `prisma`, abra o arquivo `schema.prisma`.
   - Verifique se o modelo `Task` está definido conforme abaixo:
     ```ts
     datasource db {
       provider = "postgresql"
       url      = env("DATABASE_URL")
     }
     
     generator client {
       provider = "prisma-client-js"
     }
     
     //adicione esse model ao schema.prisma
     model Task {
       id       Int      @id @default(autoincrement())
       title    String
       dueDate  DateTime
     }
     ```

## **4. Realizar a Migração do Banco de Dados**

### 4.1. **Executar a Migração**
   - No terminal, execute o seguinte comando para criar a tabela no banco de dados NeonDB:
     ```bash
     npx prisma migrate dev --name init
     ```
   - Isso irá aplicar as alterações do modelo `Task` e criar a tabela correspondente no seu banco de dados.

## **5. Implementar as Funcionalidades de API**

### 5.1. **Crie a estrutura de pastas para a API:**
   - Dentro de `src/app/`, crie a pasta `api/`.
   - Dentro de `api/`, crie a pasta `tasks/`.
   - Dentro de `tasks/`, crie o arquivo `route.ts`.
   
   A estrutura ficará assim:
   
   ```css
    src/
    └── app/
        ├── api/
        │   └── tasks/
        │       └── route.ts
   ```

### 5.2. **Crie as funcionalidades da API:**
   - Adicione o código no arquivo `src/app/api/tasks/route.ts`

     ```typescript
     import { prisma } from '@/lib/prisma';
     import { NextRequest, NextResponse } from 'next/server';
     
     export async function POST(req: NextRequest) {
       try {
         const { title, dueDate } = await req.json();
         const newTask = await prisma.task.create({
           data: {
             title,
             dueDate: new Date(dueDate),
           },
         });
         return NextResponse.json(newTask, { status: 201 });
       } catch (error) {
         return NextResponse.json({ error: 'Erro ao adicionar a tarefa' }, { status: 500 });
       }
     }
     
     export async function GET() {
       try {
         const tasks = await prisma.task.findMany();
         return NextResponse.json(tasks, { status: 200 });
       } catch (error) {
         return NextResponse.json({ error: 'Erro ao buscar as tarefas' }, { status: 500 });
       }
     }
     
     export async function DELETE(req: NextRequest) {
       try {
         const { id } = await req.json();
         const task = await prisma.task.findUnique({ where: { id } });
         if (!task) {
           return NextResponse.json({ error: 'Tarefa não encontrada' }, { status: 404 });
         }
         await prisma.task.delete({ where: { id } });
         return NextResponse.json({ message: 'Tarefa deletada com sucesso' }, { status: 200 });
       } catch (error) {
         return NextResponse.json({ error: 'Erro ao deletar a tarefa' }, { status: 500 });
       }
     }
     ```

## **6. Atualizar o Frontend**

### 6.1. **Verificar e Atualizar o Arquivo page.tsx**
   - Assegure-se de que o arquivo `src/app/page.tsx` está implementado corretamente para utilizar as funcionalidades do banco de dados:
     ```typescript
     "use client";

     import React, { useState, useEffect } from "react";

     type Task = {
       id: number;
       title: string;
       dueDate: string;
     };

     export default function Home() {
       const [tasks, setTasks] = useState<Task[]>([]);
       const [newTask, setNewTask] = useState("");
       const [dueDate, setDueDate] = useState("");

       // Função para buscar as tarefas ao carregar a página
       useEffect(() => {
         const fetchTasks = async () => {
           try {
             const response = await fetch("/api/tasks");
             if (response.ok) {
               const data = await response.json();
               setTasks(data);
             } else {
               console.error("Erro ao buscar as tarefas");
             }
           } catch (error) {
             console.error("Erro ao fazer a requisição:", error);
           }
         };

         fetchTasks();
       }, []);

       // Função para adicionar nova tarefa
       const addTask = async () => {
         if (!newTask || !dueDate) return;

         try {
           const response = await fetch("/api/tasks", {
             method: "POST",
             headers: {
               "Content-Type": "application/json",
             },
             body: JSON.stringify({ title: newTask, dueDate }),
           });

           if (response.ok) {
             const addedTask = await response.json();
             setTasks([...tasks, addedTask]);
             setNewTask("");
             setDueDate("");
           } else {
             console.error("Erro ao adicionar a tarefa");
           }
         } catch (error) {
           console.error("Erro ao enviar a requisição:", error);
         }
       };

       // Função para excluir tarefa
       const deleteTask = async (id: number) => {
         try {
           const response = await fetch("/api/tasks", {
             method: "DELETE",
             headers: {
               "Content-Type": "application/json",
             },
             body: JSON.stringify({ id }),
           });

           if (response.ok) {
             setTasks(tasks.filter((task) => task.id !== id));
           } else {
             console.error("Erro ao excluir a tarefa");
           }
         } catch (error) {
           console.error("Erro ao fazer a requisição:", error);
         }
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
                     <p className="text-gray-200">{task.title}</p>
                     <p className="text-sm text-gray-500">{new Date(task.dueDate).toLocaleString()}</p>
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

## **7. Testando a Aplicação**

### 7.1. **Iniciar o Servidor**
   - No terminal, inicie o servidor Next.js:
     ```bash
     npm run dev
     ```

### 7.2. **Testar Funcionalidades**
   - Acesse o aplicativo em `http://localhost:3000`.
   - Teste as funcionalidades de **adicionar**, **listar** e **remover** tarefas. Verifique se as tarefas estão sendo armazenadas no banco de dados no NeonDB.
