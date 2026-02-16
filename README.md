# EduBlokos MVP - Organizador de Estudos Modular

## 1. Visão Geral do Projeto

Este projeto é um MVP (Produto Viável Mínimo) de uma aplicação web para organização de materiais de estudo e planos de aula. O conceito central é a **reutilização de conteúdo**.

### Core Concepts

- **Bloko:** A menor unidade de conteúdo (ex: um parágrafo de texto, um link, uma imagem, uma anotação, um vídeo). É atômico. Pode ser reutilizado em várias Collections.
- **Collection:** Um agrupamento ordenado de Blokos. Funciona como uma "página" ou "plano de aula".
- - **Workspace:** É a "home" do usuário. É a visão agregada que contém todas as **Collections** que o usuário criou ou copiou.
- **Recombinação:** Um mesmo `Bloko` pode existir em múltiplas `Collections` diferentes.
- **Ordenação:** A ordem dos Blokos varia por Collection (ex: O Bloko A pode ser o 1º na Collection X e o 5º na Collection Y).
- **Visibilidade:** Usuários podem marcar Blokos e Collections como "Público" ou "Privado".
- - **Forking (Cópia):** Ao "salvar" um item de outro usuário, o sistema cria uma nova entrada no banco de dados pertencente ao usuário atual, mantendo um link (`OriginId`) para o item original.

---

## 2. Stack Tecnológica

### Frontend

- **Framework:** React 18+ (via Vite).
- **Linguagem:** TypeScript.
- **Estilização:** Tailwind CSS (para rapidez no MVP).
- **Gerenciamento de Estado/Data Fetching:** TanStack Query (React Query) + Axios.
- **Roteamento:** React Router DOM.
- **Ícones:** Lucide React ou Heroicons.

### Backend

- **Framework:** .NET 8 (ASP.NET Core Web API).
- **Linguagem:** C#.
- **ORM:** Entity Framework Core (Code First).
- **Auth:** JWT (JSON Web Token) nativo do .NET Identity.
- **Swagger:** Swashbuckle (para documentação da API).

### Database & Infra (Azure Free Tier)

- **Database:** Azure SQL Database (vCore Free Tier ou DTU Basic).
- **Hospedagem API:** Azure App Service (Plano F1 - Free).
- **Hospedagem Front:** Azure Static Web Apps (Plano Free).

---

## 3. Modelo de Dados (Schema Simplificado)

A IA deve seguir rigorosamente este relacionamento para permitir a recombinação e ordenação.

### Entidades Principais

**1. User**

- `Id` (Guid)
- `Username`, `Email`, `PasswordHash`

**2. Bloko**

- `Id` (Guid)
- `Content` (String/Text - suporta Markdown simples)
- `Type` (Enum: Text, Link, ImageUrl, VideoUrl)
- `IsPublic` (Bool)
- `OwnerId` (FK User) -> Quem é o dono ATUAL desta cópia.
- `OriginalBlokoId` (FK Bloko) -> ID do bloko que originou este (pode ser null se for criação original).
- `CreatedAt`

**3. Collection**

- `Id` (Guid)
- `Title` (String)
- `Description` (String)
- `IsPublic` (Bool)
- `OwnerId` (FK User)
- `OriginalCollectionId` (FK Collection) -> ID da coleção que originou esta.
- `CreatedAt`

**4. CollectionItem (Tabela de Junção/Join Entity)**

- _Esta entidade é crítica para a funcionalidade de ordenação._
- `Id` (Guid)
- `CollectionId` (FK)
- `BlokoId` (FK)
- `OrderIndex` (Int) -> Define a posição do bloco _nesta_ coleção específica.

---

## 4. Regras de Negócio (MVP)

1. **Criação:** Usuário cria Blokos soltos ou cria uma Collection e adiciona novos Blokos a ela.
2. **Visualização:**
   - Privado: Só o dono vê. Só o dono pode criar cópias.
   - Público: Todos veem (modo leitura). Todos podem criar cópias.
3. **Permissões de Edição:**
   - O usuário só pode adicionar/remover Blokos e Collections que pertencem a ele.
4. **Azure Free Tier Constraints:**
   - Evitar Storage Blob por enquanto (salvar imagens como URLs externas ou Base64 curto se for texto pequeno).
   - Banco de dados deve ser configurado leve para não estourar DTUs do tier gratuito.

5. **Cenário A: Clonar uma Collection Inteira**
   1. O sistema busca a Collection Original e seus Itens.
   2. Cria uma **Nova Collection** (Owner = Usuário Atual, OriginalId = ID da Collection Original).
   3. Itera sobre os Blokos da original:
      - Cria **Novos Blokos** (cópias do conteúdo, Owner = Usuário Atual, OriginalId = ID do Bloko Original).
   4. Cria os relacionamentos (`CollectionItem`) ligando a Nova Collection aos Novos Blokos, mantendo a ordem (`OrderIndex`) original.

6. **Cenário B: Clonar um Bloko Isolado**
   1. O sistema cria um **Novo Bloko** (Owner = Usuário Atual, OriginalId = ID do Bloko Original).
   2. O usuário deve escolher em qual de suas Collections existentes este novo bloko será inserido.

---

## 5. Estrutura de Pastas Sugerida

```txt
/src
  /Client (React App)
  /Server (NET Solution)
    /EduBlokos.API
    /EduBlokos.Core (Models/Interfaces)
    /EduBlokos.Infra (Data/EF)
```
