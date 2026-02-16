# Checklist de Desenvolvimento (Step-by-Step para IA)

## Fase 1: Setup & Infraestrutura Base

- [ ] **1.1 Setup da Solution .NET**
  - Criar solução `EduBlokos`.
  - Criar projeto Web API (.NET 8).
  - Configurar Swagger.
  - Remover "WeatherForecast".
- [ ] **1.2 Setup do React (Vite)**
  - Criar projeto React + TypeScript dentro da pasta raiz (ou pasta `/client`).
  - Instalar Tailwind CSS.
  - Configurar Proxy no `vite.config.ts` para apontar para a API .NET.

## Fase 2: Backend - Domínio e Dados

- [ ] **2.1 Modelagem das Entidades**
  - Criar classes: `User`, `Bloko`, `Collection`.
  - Criar classe de junção: `CollectionItem` (com propriedade `OrderIndex`).
- [ ] **2.2 Entity Framework Core**
  - Configurar `AppDbContext`.
  - Configurar relacionamentos (One-to-Many e Many-to-Many).
  - Configurar `OnDelete` (SetNull) - _Nota: Deletar uma coleção original NÃO deve deletar as cópias dos outros usuários._
  - **Importante:** Garantir que o delete de uma Collection _não_ delete os Blokos (apenas remova a associação).
- [ ] **2.3 Migrations e Database**
  - Configurar Connection String (LocalDB para dev, Azure SQL para prod).
  - Criar e aplicar primeira Migration.

## Fase 3: Backend - API e Regras

- [ ] **3.1 Auth Simplificada**
  - Endpoint `/login` e `/register`.
  - Gerar Token JWT.
  - Middleware de Autenticação.
- [ ] **3.2 CRUD de Blokos**
  - `POST /api/blokos` (Criar)
  - `GET /api/blokos` (Listar meus blokos)
  - `PUT /api/blokos/{id}` (Editar conteúdo)
- [ ] **3.3 CRUD de Collections**
  - `POST /api/collections` (Criar container vazio)
  - `GET /api/collections/{id}` (Retornar detalhes + lista de Blokos ordenada por `OrderIndex`).
- [ ] **3.4 Manipulação da Coleção (Core Feature)**
  - `POST /api/collections/{id}/add-bloko` (Recebe `blokoId` e adiciona ao final da lista).
  - `PUT /api/collections/{id}/reorder` (Recebe lista de IDs na nova ordem e atualiza `OrderIndex`).

## Fase 4: Frontend - Integração MVP

- [ ] **4.1 Estrutura de UI**
  - Criar Layout base (Navbar, Sidebar).
  - Configurar Contexto de Auth (Login/Logout).
- [ ] **4.2 Tela: Meus Blokos (Biblioteca)**
  - Listagem simples de cartões com os blokos criados.
  - Modal/Form para criar novo Bloko rápido.
- [ ] **4.3 Tela: Collection**
  - Visualizar Título da Collection.
  - Listar os Blokos desta collection em ordem.
  - Botão "Adicionar Bloko Existente" (abre um modal para selecionar da biblioteca).
  - Botão "Remover Bloko" (remove da lista, mantém no banco).
- [ ] **4.4 Tela: Dashboard**
  - Listagem das Collections do usuário.

## Fase 5: Azure Deployment (Free Tier)

- [ ] **5.1 Preparação**
  - Criar script SQL para gerar tabelas no Azure SQL (caso migrations falhem na pipeline).
- [ ] **5.2 Backend Publish**
  - Configurar Profile de publicação para Azure App Service (Windows/Linux F1 Free).
- [ ] **5.3 Frontend Publish**
  - Build do React.
  - Configurar deploy para Azure Static Web Apps ou servir arquivos estáticos via wwwroot do .NET (mais simples para MVP único).
