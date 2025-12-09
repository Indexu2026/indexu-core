# Indexu Core – Backend Architecture

> 📂 Este documento descreve a arquitetura **alta nível** do backend do projeto Indexu Core.  
> Não contém segredos, credenciais ou detalhes sensíveis.

---

## 1. Visão Geral

O backend do **Indexu Core** é uma API construída em **Node.js + TypeScript**, focada em:

- gerenciamento de usuários e perfis dimensionais;
- processamento e análise de mídia (vídeos);
- orquestração de módulos de IA (dimensionais, reputação, participação, etc.);
- integração futura com camadas de web3 e tokenização.

A arquitetura segue um modelo **modular**:

- cada domínio (ex: `profile`, `media`, `userdimensions`) fica em um módulo próprio;
- controllers, services, routes e dtos são separados por responsabilidade;
- filas e workers são usados para tarefas pesadas e assíncronas.

---

## 2. Tech Stack (alto nível)

- **Linguagem:** Node.js + TypeScript  
- **Framework HTTP:** Express  
- **ORM:** Prisma  
- **Banco de Dados:** PostgreSQL  
- **Filas & Jobs:** BullMQ + Redis  
- **Armazenamento de Mídia:** Object Storage compatível com S3  
- **Autenticação:** JWT + middleware de sessão  
- **Estilo de Arquitetura:** modular, orientada a domínio (Domain-Oriented Modules)

---

## 3. Estrutura de Pastas (simplificada)

```bash
src/
  app.ts              # Inicialização da aplicação
  server.ts           # Bootstrap do servidor HTTP
  typings.ts          # Tipagem global

  auth/               # Autenticação
  config/             # Configurações (Redis, filas, etc.)
  db/                 # Cliente Prisma
  middlewares/        # Middlewares de autenticação e utilitários

  modules/            # Módulos de domínio
    profile/          # Perfil do usuário
    media/            # Upload, filas e processamento de vídeo
    userdimensions/   # Motor dimensional IND / EXD / U
    networkai/        # Agentes de rede de IA
    feed/             # Candidatos de feed
    reputation/       # Reputação do usuário
    participation/    # Eventos de participação
    tokenengine/      # Engine de tokens (futuro / web3)
    userai/           # Perfil de IA por usuário
    web3/             # Integrações com blockchain (alto nível)

  onboarding/         # Fluxo de onboarding do usuário
  upload/             # Upload de arquivos + integração com storage
  user/               # Controladores de usuário
  workers/            # Workers (ex: processamento de vídeo)
  utils/              # Utilitários (jwt, hash, storage, etc.)
  types/              # Tipagens auxiliares
💡 Observação: essa estrutura é intencionalmente genérica e não expõe detalhes internos específicos.

4. Fluxo de Requisição (API HTTP)
O cliente (app/web) faz uma requisição HTTP para o backend.

A requisição passa por:

middleware de logging (opcional),

middleware de autenticação (quando necessário).

O request é encaminhado para o controller do módulo correspondente.

O controller chama um service, onde fica a regra de negócio.

O service conversa com:

Prisma (PostgreSQL),

filas (BullMQ),

storage de mídia,

outros módulos internos, quando necessário.

A resposta é devolvida ao cliente em formato JSON.

5. Pipeline de Mídia (Visão Geral)
O pipeline de mídia foi projetado para ser assíncrono e escalável:

O usuário solicita a criação de um upload de vídeo.

O backend gera uma URL de upload e registra a mídia no banco com status uploading.

Após o upload, um worker é acionado via fila para:

processar o vídeo,

extrair frames,

atualizar metadados da mídia.

A análise da mídia (ex.: futuro pipeline de IA multimodal) é feita em etapa separada.

Ao final, o status da mídia é atualizado para ready ou failed.

Nenhum endpoint público, segredo, bucket ou URL real são expostos neste documento.

6. Modelo de Dados (Resumo Prisma)
A definição dos modelos é feita com Prisma.
Exemplo simplificado (não sensível):

ts
Copiar código
model User {
  id        String   @id @default(uuid())
  email     String   @unique
  name      String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  medias Media[]
}

model Media {
  id        String      @id @default(cuid())
  userId    String
  type      String
  status    MediaStatus @default(uploading)
  createdAt DateTime    @default(now())
  updatedAt DateTime    @updatedAt

  user   User          @relation(fields: [userId], references: [id])
  frames MediaFrame[]
}

enum MediaStatus {
  uploading
  processing
  ready
  failed
  deleted
}
Aqui você mostra a ideia da modelagem sem expor campos internos mais sensíveis ou lógicas específicas.

7. Segurança e Boas Práticas
Este repositório:

não inclui arquivos .env ou credenciais;

não expõe URLs internas de serviços;

segue o princípio de configuração via variáveis de ambiente;

trata o banco, storage e serviços externos como dependências configuráveis, não hardcoded.

Exemplo de boas práticas (genérico):

bash
Copiar código
# .env (não versionado)
DATABASE_URL="postgresql://user:password@host:5432/database"
REDIS_URL="redis://host:6379"
STORAGE_ENDPOINT="https://storage-provider"
8. Roadmap Técnico (alto nível)
 Consolidar módulo de análise de mídia (IA multimodal).

 Expandir o motor dimensional (IND / EXD / U) com mais sinais.

 Integrar camada de reputação com o feed inteligente.

 Evoluir token engine para interação com web3.

 Refinar documentação pública sem expor detalhes sensíveis.

9. Licença & Aviso
Este projeto está em desenvolvimento ativo.
A arquitetura aqui descrita é alto nível e pode evoluir ao longo do tempo.