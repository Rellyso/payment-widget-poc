# 🎯 Fluxo de Deploy - CloudFront Setup

```
┌─────────────────────────────────────────────────────────────────────┐
│                    🚀 PAYMENT WIDGET - DEPLOY FLOW                   │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                         1️⃣ PRIMEIRA VEZ                              │
└─────────────────────────────────────────────────────────────────────┘

    📖 Ler Documentação
    │
    ├─► docs/CLOUDFRONT-SETUP.md     (Guia completo)
    └─► SETUP-COMPLETO.md             (Resumo executivo)
    
    ⬇️
    
    ⚙️ Configurar CloudFront
    │
    └─► ./setup-cloudfront.sh production
        │
        ├─► ✅ Cria Origin Access Identity (OAI)
        ├─► ✅ Cria Distribuição CloudFront
        ├─► ✅ Configura Bucket S3
        ├─► ✅ Atualiza deploy.sh com Distribution ID
        ├─► ✅ Aguarda deploy (15-30 min)
        └─► ✅ Gera cloudfront-setup-report.txt
    
    ⬇️
    
    📄 Verificar Resultado
    │
    ├─► .oai-id                       (ID do OAI)
    ├─► .distribution-id              (ID da distribuição)
    ├─► .cloudfront-domain            (Domain Name)
    └─► cloudfront-setup-report.txt   (Relatório completo)


┌─────────────────────────────────────────────────────────────────────┐
│                         2️⃣ DEPLOY REGULAR                            │
└─────────────────────────────────────────────────────────────────────┘

    🔨 Build + Deploy
    │
    └─► ./deploy.sh production
        │
        ├─► 🏗️  Build SDK, CDN, Bootstrap
        ├─► 📦 Upload para S3
        │   ├─ widget-bootstrap.v1.min.js  (~5KB)   Cache: 5min
        │   ├─ widget.v1.min.js            (~427KB) Cache: 1yr
        │   └─ widget.v1.min.css           (~29KB)  Cache: 1yr
        │
        ├─► 🔄 Invalida CloudFront
        ├─► ✅ Testa URLs
        └─► 📊 Gera deploy-report.json


┌─────────────────────────────────────────────────────────────────────┐
│                         3️⃣ TESTAR                                    │
└─────────────────────────────────────────────────────────────────────┘

    🧪 Testes Automatizados
    │
    └─► examples/cloudfront-test.html
        │
        ├─► ✅ Testa Bootstrap carrega
        ├─► ✅ Testa Bundle carrega
        ├─► ✅ Testa CSS carrega
        ├─► ✅ Testa Shadow DOM cria
        ├─► ✅ Testa Modal abre/fecha
        └─► ✅ Testa Estilos aplicam


┌─────────────────────────────────────────────────────────────────────┐
│                         4️⃣ STAGING                                   │
└─────────────────────────────────────────────────────────────────────┘

    🧪 Ambiente de Testes
    │
    └─► ./deploy.sh staging
        │
        ├─► 📦 Upload para S3 Staging
        │   └─ cartao-simples-widget-staging
        │
        └─► 🔗 URLs diretas (sem CloudFront)
            ├─ https://cartao-simples-widget-staging.s3.us-east-1.amazonaws.com/...
            └─ Testes rápidos sem propagação CDN


┌─────────────────────────────────────────────────────────────────────┐
│                    📁 ESTRUTURA DE ARQUIVOS                          │
└─────────────────────────────────────────────────────────────────────┘

payment-widget-poc-v2/
│
├── 🆕 setup-cloudfront.sh          ⭐ Script de configuração automática
├── 🆕 SETUP-COMPLETO.md            ⭐ Resumo executivo
│
├── deploy.sh                       Atualizado com Distribution ID
├── README.md                       Atualizado com Deploy Rápido
├── .gitignore                      Atualizado com arquivos temporários
│
├── docs/
│   ├── 🆕 CLOUDFRONT-SETUP.md      ⭐ Guia completo CloudFront
│   ├── 🆕 RESUMO-CLOUDFRONT.md     ⭐ Resumo CloudFront
│   ├── DEPLOY-GUIDE.md             Atualizado com pré-requisitos
│   ├── DOCS-INDEX.md               Atualizado com novas docs
│   ├── GUIA-DEPLOY-CDN.md
│   ├── QUICK-START.md
│   └── ...
│
├── examples/
│   ├── cloudfront-test.html        Teste de CDN
│   ├── test-staging.html           Teste staging
│   └── ...
│
└── [após setup]
    ├── .oai-id                     ID do OAI
    ├── .distribution-id            ID da distribuição
    ├── .cloudfront-domain          Domain do CloudFront
    └── cloudfront-setup-report.txt Relatório completo


┌─────────────────────────────────────────────────────────────────────┐
│                    🎯 COMANDOS PRINCIPAIS                            │
└─────────────────────────────────────────────────────────────────────┘

# 1. Primeira vez (Setup CloudFront)
./setup-cloudfront.sh production

# 2. Deploy regular
./deploy.sh production

# 3. Deploy staging
./deploy.sh staging

# 4. Ver domain CloudFront
cat .cloudfront-domain

# 5. Invalidar cache
aws cloudfront create-invalidation \
  --distribution-id $(cat .distribution-id) \
  --paths "/*"

# 6. Verificar status
aws cloudfront get-distribution \
  --id $(cat .distribution-id) | jq -r '.Distribution.Status'


┌─────────────────────────────────────────────────────────────────────┐
│                    📊 CACHE POLICIES                                 │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────┬──────────┬────────────────────────┐
│ Arquivo                 │ Cache    │ Motivo                 │
├─────────────────────────┼──────────┼────────────────────────┤
│ widget-bootstrap.v1.min │ 5 min    │ Permite atualizações   │
│ widget.v1.min.js        │ 1 ano    │ Versionado (imutável)  │
│ widget.v1.min.css       │ 1 ano    │ Versionado (imutável)  │
└─────────────────────────┴──────────┴────────────────────────┘


┌─────────────────────────────────────────────────────────────────────┐
│                    💰 CUSTOS ESTIMADOS                               │
└─────────────────────────────────────────────────────────────────────┘

100.000 pageviews/mês:

┌──────────────────┬────────────┐
│ Item             │ Custo/mês  │
├──────────────────┼────────────┤
│ Data Transfer    │ ~$0.85     │
│ HTTPS Requests   │ ~$1.00     │
│ S3 Storage       │ ~$0.02     │
│ S3 Requests      │ ~$0.04     │
├──────────────────┼────────────┤
│ 💵 TOTAL         │ ~$2-3      │
└──────────────────┴────────────┘

AWS Free Tier: 1 TB/mês grátis no 1º ano! 🎉


┌─────────────────────────────────────────────────────────────────────┐
│                    ✅ CHECKLIST                                      │
└─────────────────────────────────────────────────────────────────────┘

Antes do Setup:
☐ AWS CLI configurado (`aws configure`)
☐ jq instalado (`brew install jq`)
☐ Permissões: S3 + CloudFront
☐ Li CLOUDFRONT-SETUP.md

Durante Setup:
☐ Executar `./setup-cloudfront.sh production`
☐ Aguardar conclusão (15-30 min)
☐ Verificar mensagem de sucesso

Após Setup:
☐ Ler cloudfront-setup-report.txt
☐ Anotar .cloudfront-domain
☐ Executar `./deploy.sh production`
☐ Testar URLs (curl -I)
☐ Testar modal (cloudfront-test.html)
☐ Verificar console sem erros
☐ Commit e push


┌─────────────────────────────────────────────────────────────────────┐
│                    🚨 TROUBLESHOOTING                                │
└─────────────────────────────────────────────────────────────────────┘

┌────────────────────────┬───────────────────────────────────────┐
│ Problema               │ Solução                               │
├────────────────────────┼───────────────────────────────────────┤
│ Bucket não existe      │ ./deploy.sh antes do setup            │
│ AWS CLI não configurado│ aws configure                         │
│ jq não instalado       │ brew install jq                       │
│ CloudFront não responde│ Aguardar 30 min (normal)              │
│ Cache desatualizado    │ aws cloudfront create-invalidation    │
│ 404 nos arquivos       │ Verificar deploy.sh executou          │
└────────────────────────┴───────────────────────────────────────┘


┌─────────────────────────────────────────────────────────────────────┐
│                    📚 DOCUMENTAÇÃO                                   │
└─────────────────────────────────────────────────────────────────────┘

Guia Rápido:
├─► SETUP-COMPLETO.md              Resumo executivo
├─► docs/CLOUDFRONT-SETUP.md       Guia completo CloudFront
└─► docs/RESUMO-CLOUDFRONT.md      Resumo CloudFront

Deploy:
├─► docs/DEPLOY-GUIDE.md           Deploy passo a passo
└─► docs/GUIA-DEPLOY-CDN.md        Infraestrutura completa

Referência:
├─► docs/DOCS-INDEX.md             Índice completo
├─► docs/QUICK-START.md            Quick start
└─► README.md                      Documentação principal


┌─────────────────────────────────────────────────────────────────────┐
│                    🎉 PRONTO!                                        │
└─────────────────────────────────────────────────────────────────────┘

Execute:  ./setup-cloudfront.sh production
Aguarde:  15-30 minutos
Deploy:   ./deploy.sh production
Teste:    examples/cloudfront-test.html

🚀 Boa sorte com o deploy!
```
