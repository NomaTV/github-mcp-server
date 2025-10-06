# NomaTV - Sistema de Gerenciamento IPTV

## Visão Geral

O **NomaTV** é um sistema completo de gerenciamento para provedores de IPTV, desenvolvido em PHP com SQLite. Oferece uma interface web moderna para administração de revendedores, provedores, clientes e controle financeiro.

## Arquitetura do Sistema

### Estrutura Hierárquica
```
Administrador
├── Master Revendedor
│   └── Sub-Revendedor
│       └── Cliente Final
└── Master Revendedor
    └── Sub-Revendedor
        └── Cliente Final
```

### Componentes Principais

#### Backend (PHP + SQLite)
- **API RESTful** para todas as operações
- **Autenticação baseada em sessão**
- **Controle de permissões hierárquico**
- **Banco de dados SQLite** para persistência

#### Frontend (HTML/CSS/JavaScript)
- **Interface responsiva** com Bootstrap 5
- **SPA (Single Page Application)**
- **Dashboard interativo** com gráficos
- **Gerenciamento completo** via modais

#### Servidor (Node.js)
- **Servidor proxy** para API PHP
- **Servidor estático** para arquivos frontend
- **CORS habilitado** para desenvolvimento

## Funcionalidades

### 👥 Gestão de Revendedores
- Cadastro de revendedores (Admin, Master, Sub)
- Controle hierárquico de permissões
- Definição de planos e valores
- Relatórios de performance

### 🖥️ Gestão de Provedores
- Cadastro de servidores IPTV
- Vinculação a revendedores
- Monitoramento de status
- Controle de acesso

### 📺 Gestão de Ativos
- Controle de client_ids
- Ativação/desativação em lote
- Bloqueio de acesso
- Histórico de atividade

### 💰 Gestão Financeira
- Controle de receita por revendedor
- Relatórios financeiros
- Previsões e projeções
- Histórico de pagamentos

### 🔒 Segurança
- Alteração segura de senhas
- Verificação de força de senha
- Logs de auditoria
- Controle de acesso por IP

### 📊 Relatórios e Analytics
- Dashboard executivo
- Relatórios financeiros
- Análise de crescimento
- Métricas de performance

## Estrutura do Banco de Dados

### Tabelas Principais

#### `revendedores`
```sql
CREATE TABLE revendedores (
    id_revendedor INTEGER PRIMARY KEY AUTOINCREMENT,
    nome TEXT NOT NULL,
    usuario TEXT UNIQUE NOT NULL,
    senha TEXT NOT NULL,
    master TEXT CHECK(master IN ('admin', 'sim', 'nao')) DEFAULT 'nao',
    plano TEXT,
    valor_ativo REAL,
    valor_mensal REAL,
    ativo INTEGER DEFAULT 1,
    criado_em DATETIME DEFAULT CURRENT_TIMESTAMP,
    atualizado_em DATETIME DEFAULT CURRENT_TIMESTAMP,
    parent_id INTEGER,
    ultra_master_id INTEGER,
    data_bloqueio DATETIME,
    logo_filename TEXT,
    FOREIGN KEY (parent_id) REFERENCES revendedores(id_revendedor)
);
```

#### `provedores`
```sql
CREATE TABLE provedores (
    id_provedor INTEGER PRIMARY KEY AUTOINCREMENT,
    nome TEXT NOT NULL,
    dns TEXT NOT NULL,
    usuario TEXT,
    senha TEXT,
    id_revendedor INTEGER NOT NULL,
    ativo INTEGER DEFAULT 1,
    criado_em DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (id_revendedor) REFERENCES revendedores(id_revendedor)
);
```

#### `client_ids`
```sql
CREATE TABLE client_ids (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    client_id TEXT UNIQUE NOT NULL,
    provedor_id INTEGER,
    id_revendedor INTEGER NOT NULL,
    usuario TEXT,
    senha TEXT,
    primeira_conexao DATETIME,
    ultima_atividade DATETIME,
    ativo INTEGER DEFAULT 1,
    bloqueado INTEGER DEFAULT 0,
    ip TEXT,
    user_agent TEXT,
    criado_em DATETIME DEFAULT CURRENT_TIMESTAMP,
    atualizado_em DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (provedor_id) REFERENCES provedores(id_provedor),
    FOREIGN KEY (id_revendedor) REFERENCES revendedores(id_revendedor)
);
```

#### `auditoria`
```sql
CREATE TABLE auditoria (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    id_revendedor INTEGER,
    acao TEXT NOT NULL,
    detalhes TEXT,
    ip TEXT,
    user_agent TEXT,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (id_revendedor) REFERENCES revendedores(id_revendedor)
);
```

## Endpoints da API

### Autenticação
- `POST /api/auth.php` - Login
- `POST /api/auth.php?action=logout` - Logout
- `GET /api/verificar_sessao.php` - Verificar sessão

### Revendedores
- `GET /api/revendedores.php` - Listar revendedores
- `POST /api/revendedores.php` - Criar revendedor
- `PUT /api/revendedores.php` - Atualizar revendedor
- `DELETE /api/revendedores.php?id={id}` - Excluir revendedor

### Provedores
- `GET /api/provedores.php` - Listar provedores
- `POST /api/provedores.php` - Criar provedor
- `PUT /api/provedores.php` - Atualizar provedor
- `DELETE /api/provedores.php?id={id}` - Excluir provedor

### Ativos (Client IDs)
- `GET /api/client_ids.php` - Listar ativos
- `PUT /api/client_ids.php` - Atualizar status
- `POST /api/client_ids.php?action=bulk` - Ações em lote

### Financeiro
- `GET /api/financeiro.php` - Dados financeiros
- `GET /api/relatorios.php?tipo=financeiro` - Relatórios financeiros

### Segurança
- `POST /api/seguranca.php?action=alterar_senha_admin` - Alterar senha
- `POST /api/seguranca.php?action=verificar_forca_senha` - Verificar força
- `GET /api/seguranca.php` - Info do servidor

### Relatórios
- `GET /api/relatorios.php?tipo=dashboard` - Dashboard
- `GET /api/relatorios.php?tipo=revendedores` - Revendedores
- `GET /api/relatorios.php?tipo=provedores` - Provedores
- `GET /api/relatorios.php?tipo=atividade` - Atividade
- `GET /api/relatorios.php?tipo=crescimento` - Crescimento
- `GET /api/relatorios.php?tipo=completo` - Relatório completo

### Configurações
- `GET /api/configuracoes.php` - Buscar configurações
- `PUT /api/configuracoes.php` - Salvar configurações
- `POST /api/configuracoes.php?action=criar_backup` - Criar backup
- `GET /api/configuracoes.php?action=listar_backups` - Listar backups
- `GET /api/configuracoes.php?action=baixar_backup&filename={file}` - Baixar backup
- `POST /api/configuracoes.php?action=restaurar_backup` - Restaurar backup

### Branding
- `GET /api/branding/get.php` - Verificar logo
- `POST /api/branding/upload.php` - Upload de logo
- `DELETE /api/branding/delete.php` - Remover logo

### Utilitários
- `GET /api/ips.php` - Controle de IPs
- `GET /api/logs.php` - Logs de auditoria
- `GET /api/permissoes.php` - Gerenciar permissões
- `GET /api/stats.php` - Estatísticas do dashboard
- `POST /api/cleanup.php` - Limpeza automática
- `GET /api/rede_revendedor.php` - Rede de revendedores

## Instalação e Configuração

### Pré-requisitos
- PHP 7.4+
- SQLite3
- Node.js 14+
- Servidor web (Apache/Nginx)

### Instalação

1. **Clone o repositório**
```bash
git clone https://github.com/NomaTV/painel-backend-corrigido.git
cd painel-backend-corrigido
```

2. **Configure o banco de dados**
```bash
cd vaiiii/api
php db_installer.php
```

3. **Instale dependências do Node.js**
```bash
npm install
```

4. **Inicie o servidor**
```bash
node server.js
```

5. **Acesse o sistema**
```
http://localhost:3000
```

### Configuração Inicial

Após a instalação, acesse com as credenciais padrão:
- **Usuário:** admin
- **Senha:** admin

⚠️ **Importante:** Altere a senha padrão imediatamente após o primeiro acesso.

## Desenvolvimento

### Estrutura de Arquivos
```
vaiiii/
├── admin.html              # Painel administrativo
├── index.html              # Página de login
├── sub_revendedor.html     # Painel sub-revendedor
├── api.js                  # Cliente API JavaScript
├── server.js               # Servidor Node.js
├── README.MD               # Esta documentação
└── api/                    # Backend PHP
    ├── auth.php
    ├── revendedores.php
    ├── provedores.php
    ├── client_ids.php
    ├── financeiro.php
    ├── relatorios.php
    ├── seguranca.php
    ├── configuracoes.php
    ├── branding/
    │   ├── get.php
    │   ├── upload.php
    │   └── delete.php
    ├── config/
    │   ├── database_sqlite.php
    │   └── api/
    │       ├── config/
    │       │   └── database_sqlite.php
    │       ├── db.db
    │       ├── helpers/
    │       │   └── auth_helper.php
    │       └── logos/
    ├── ips.php
    ├── logs.php
    ├── permissoes.php
    ├── stats.php
    ├── validar_login.php
    ├── verificar_provedor.php
    ├── verificar_sessao.php
    ├── cleanup.php
    ├── rede_revendedor.php
    └── db.db
```

### Convenções de Código

#### PHP
- PSR-12 para formatação
- Type hints quando possível
- Tratamento de exceções
- Respostas JSON padronizadas

#### JavaScript
- ES6+ features
- Async/await para operações assíncronas
- Tratamento de erros
- Separação de responsabilidades

#### HTML/CSS
- Bootstrap 5 para componentes
- CSS customizado mínimo
- Acessibilidade
- Responsividade

## Segurança

### Medidas Implementadas
- **Hashing de senhas** com password_hash()
- **Sessões seguras** com regeneração de ID
- **Validação de entrada** em todos os endpoints
- **Logs de auditoria** para todas as ações
- **Controle de permissões** baseado em hierarquia
- **Proteção contra SQL Injection** com prepared statements
- **Headers de segurança** (CORS, CSP)

### Recomendações Adicionais
- Use HTTPS em produção
- Configure firewall
- Monitore logs regularmente
- Faça backups automáticos
- Mantenha dependências atualizadas

## Monitoramento e Manutenção

### Limpeza Automática
O sistema inclui um script de limpeza automática (`cleanup.php`) que:
- Inativa client_ids sem atividade há 7 dias
- Remove registros inativos há 60 dias
- Limpa logs antigos (90 dias)
- Remove arquivos órfãos
- Otimiza o banco de dados

**Recomendação:** Configure um cron job diário:
```bash
0 3 * * * /usr/bin/php /path/to/cleanup.php
```

### Backups
O sistema suporta backup e restauração completos:
- Backup do banco de dados + arquivos
- Restauração com verificação de integridade
- Lista de backups disponíveis
- Download de backups específicos

### Logs e Auditoria
- Todas as ações são registradas
- Filtros por usuário, ação e data
- Exportação para análise
- Retenção configurável

## Troubleshooting

### Problemas Comuns

#### Erro de conexão com banco de dados
```
Verifique se o arquivo db.db existe e tem permissões corretas
Execute: chmod 664 db.db
```

#### Erro 404 nas APIs
```
Verifique se o servidor Node.js está rodando
Verifique se as URLs estão corretas
```

#### Problemas de CORS
```
Verifique se o servidor permite origens corretas
Configure headers adequados
```

#### Erro de upload de logo
```
Verifique permissões da pasta uploads/logos/
Verifique tamanho e formato do arquivo
```

### Debug Mode
Para ativar o modo debug, adicione ao `server.js`:
```javascript
process.env.DEBUG = 'true';
```

## Contribuição

1. Fork o projeto
2. Crie uma branch para sua feature (`git checkout -b feature/nova-feature`)
3. Commit suas mudanças (`git commit -am 'Adiciona nova feature'`)
4. Push para a branch (`git push origin feature/nova-feature`)
5. Abra um Pull Request

## Licença

Este projeto está sob a licença MIT. Veja o arquivo `LICENSE` para mais detalhes.

## Suporte

Para suporte técnico, entre em contato:
- **Email:** suporte@nomatv.com
- **Discord:** [Servidor NomaTV](https://discord.gg/nomatv)
- **Documentação:** [Wiki do Projeto](https://github.com/NomaTV/painel-backend-corrigido/wiki)

## Changelog

### v4.5 - [Data Atual]
- ✨ Sistema de branding personalizado
- ✨ Relatórios avançados
- ✨ Controle de rede revendedor
- 🔒 Melhorias de segurança
- 🐛 Correções de bugs

### v4.2 - [Data Anterior]
- 📊 Dashboard interativo
- 💰 Sistema financeiro completo
- 👥 Hierarquia de revendedores
- 🔄 API RESTful completa

### v4.0 - [Lançamento Inicial]
- 🖥️ Interface administrativa
- 📺 Gestão de provedores e ativos
- 🔐 Sistema de autenticação
- 📱 Design responsivo

---

**NomaTV** - Sistema profissional de gerenciamento IPTV 🚀