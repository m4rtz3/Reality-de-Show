# 🎭 Sistema Reality Show API

Sistema completo de gerenciamento de reality shows com **Node.js**, **Express** e **MongoDB**. Gerencie participantes, prêmios, audiência e estatísticas detalhadas.

## 📋 Pré-requisitos

- [Play with Docker](https://labs.play-with-docker.com/) (1 instância)
- Navegador web com suporte a cURL

## 🚀 Instalação

### 📦 Instância: MongoDB + Node.js

#### 1. **Configurar Repositórios e Instalar Dependências:**

```bash
echo 'http://dl-cdn.alpinelinux.org/alpine/v3.9/main' >> /etc/apk/repositories
echo 'http://dl-cdn.alpinelinux.org/alpine/v3.9/community' >> /etc/apk/repositories
apk update
apk add mongodb mongodb-tools nodejs npm
```

#### 2. **Iniciar o MongoDB:**

```bash
mkdir ~/mongodb
mongod --dbpath /root/mongodb --fork --logpath /dev/null --bind_ip_all
```

> ✅ Aguarde alguns segundos para o MongoDB inicializar completamente.

#### 3. **Criar Estrutura do Projeto:**

```bash
cd ~
mkdir reality-app
cd reality-app
```

#### 4. **Criar Arquivo de Dados (`reality_shows.json`):**

```bash
cat > reality_shows.json << 'EOF'
[
  {
    "nome": "Big Brother Brasil 2025",
    "emissora": {
      "nome": "Globo",
      "pontos_audiencia": 85
    },
    "participantes": [
      {"nome": "Ana Costa", "idade": 25, "premios": [{"descricao": "Tablet", "valor": 2000, "data_recebimento": "2025-01-10"}]},
      {"nome": "Bruno Silva", "idade": 30, "premios": []},
      {"nome": "Carla Mendes", "idade": 28, "premios": [{"descricao": "Smart TV", "valor": 3500, "data_recebimento": "2025-02-05"}]},
      {"nome": "Daniel Souza", "idade": 32, "premios": []},
      {"nome": "Elena Rocha", "idade": 27, "premios": [{"descricao": "Notebook", "valor": 4000, "data_recebimento": "2025-02-20"}]},
      {"nome": "Felipe Lima", "idade": 29, "premios": []},
      {"nome": "Gabriela Santos", "idade": 26, "premios": [{"descricao": "Viagem", "valor": 8000, "data_recebimento": "2025-03-01"}]},
      {"nome": "Hugo Alves", "idade": 31, "premios": []},
      {"nome": "Isabela Dias", "idade": 24, "premios": []},
      {"nome": "João Oliveira", "idade": 33, "premios": [{"descricao": "Carro", "valor": 80000, "data_recebimento": "2025-03-15"}]}
    ]
  },
  {
    "nome": "MasterChef Brasil",
    "emissora": {
      "nome": "Band",
      "pontos_audiencia": 72
    },
    "participantes": [
      {"nome": "Lucas Martins", "idade": 35, "premios": [{"descricao": "Kit Cozinha", "valor": 5000, "data_recebimento": "2025-01-20"}]},
      {"nome": "Marina Castro", "idade": 29, "premios": []},
      {"nome": "Nicolas Ferreira", "idade": 27, "premios": []},
      {"nome": "Olivia Pinto", "idade": 31, "premios": [{"descricao": "Curso Gastronomia", "valor": 12000, "data_recebimento": "2025-02-10"}]},
      {"nome": "Paulo Gomes", "idade": 28, "premios": []},
      {"nome": "Rafaela Nunes", "idade": 30, "premios": []},
      {"nome": "Sergio Barros", "idade": 34, "premios": [{"descricao": "Prêmio Final", "valor": 200000, "data_recebimento": "2025-03-20"}]},
      {"nome": "Tatiana Cruz", "idade": 26, "premios": []},
      {"nome": "Vitor Ribeiro", "idade": 32, "premios": []},
      {"nome": "Wesley Moura", "idade": 29, "premios": [{"descricao": "Estágio", "valor": 15000, "data_recebimento": "2025-02-25"}]}
    ]
  },
  {
    "nome": "A Fazenda 2025",
    "emissora": {
      "nome": "Record",
      "pontos_audiencia": 68
    },
    "participantes": [
      {"nome": "Amanda Reis", "idade": 26, "premios": []},
      {"nome": "Bernardo Lopes", "idade": 33, "premios": [{"descricao": "Moto", "valor": 25000, "data_recebimento": "2025-01-30"}]},
      {"nome": "Camila Teixeira", "idade": 28, "premios": []},
      {"nome": "Diego Cardoso", "idade": 30, "premios": []},
      {"nome": "Eduarda Morais", "idade": 25, "premios": [{"descricao": "Jet Ski", "valor": 35000, "data_recebimento": "2025-02-15"}]},
      {"nome": "Fabio Araujo", "idade": 31, "premios": []},
      {"nome": "Giovana Silva", "idade": 27, "premios": []},
      {"nome": "Henrique Sousa", "idade": 29, "premios": [{"descricao": "Prêmio Fazendeiro", "valor": 150000, "data_recebimento": "2025-03-10"}]},
      {"nome": "Iris Monteiro", "idade": 24, "premios": []},
      {"nome": "Jorge Cavalcanti", "idade": 32, "premios": [{"descricao": "Carro", "valor": 90000, "data_recebimento": "2025-03-18"}]}
    ]
  }
]
EOF
```

#### 5. **Importar Dados para o MongoDB:**

```bash
mongoimport --db reality --collection shows --file reality_shows.json --jsonArray
```

Verifique a importação:

```bash
mongo reality --eval "db.shows.count()"
```

Resultado esperado: `3`

#### 6. **Inicializar Projeto Node.js:**

```bash
npm init -y
npm install --save express mongodb@4.0.0
```

#### 7. **Criar o Servidor (`server.js`):**

```bash
cat > server.js << 'EOF'
const express = require('express');
const { MongoClient } = require('mongodb');

const app = express();
app.use(express.json());

const url = 'mongodb://localhost:27017';
let db;

// Middleware para tratamento de erros
const asyncHandler = (fn) => (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
};

// GET /premios - Reality shows, participantes e prêmios detalhados
app.get('/premios', asyncHandler(async (req, res) => {
    const shows = await db.collection('shows').find({}).toArray();
    
    if (!shows || shows.length === 0) {
        return res.status(404).json({
            sucesso: false,
            mensagem: 'Nenhum reality show encontrado',
            dados: []
        });
    }
    
    const resultado = shows.map(show => {
        const participantesComPremios = show.participantes.filter(p => p.premios && p.premios.length > 0);
        const totalPremiosShow = show.participantes.reduce((total, p) => {
            return total + (p.premios || []).reduce((sum, premio) => sum + premio.valor, 0);
        }, 0);
        
        return {
            reality_show: show.nome,
            emissora: show.emissora.nome,
            total_participantes: show.participantes.length,
            participantes_premiados: participantesComPremios.length,
            valor_total_premios: totalPremiosShow,
            participantes: show.participantes.map(p => ({
                nome: p.nome,
                idade: p.idade,
                quantidade_premios: (p.premios || []).length,
                valor_total_ganho: (p.premios || []).reduce((sum, premio) => sum + premio.valor, 0),
                premios: (p.premios || []).map(premio => ({
                    descricao: premio.descricao,
                    valor: premio.valor,
                    data_recebimento: premio.data_recebimento
                }))
            })).sort((a, b) => b.valor_total_ganho - a.valor_total_ganho)
        };
    });
    
    const resumoGeral = {
        total_reality_shows: shows.length,
        total_participantes: shows.reduce((sum, s) => sum + s.participantes.length, 0),
        valor_total_distribuido: resultado.reduce((sum, r) => sum + r.valor_total_premios, 0),
        total_premios_distribuidos: shows.reduce((sum, s) => 
            sum + s.participantes.reduce((pSum, p) => pSum + (p.premios || []).length, 0), 0
        )
    };
    
    res.json({
        sucesso: true,
        resumo: resumoGeral,
        reality_shows: resultado
    });
}));

// GET /idade/:nome_reality - Participante mais novo e mais velho com estatísticas
app.get('/idade/:nome_reality', asyncHandler(async (req, res) => {
    const nomeReality = decodeURIComponent(req.params.nome_reality);
    
    const show = await db.collection('shows').findOne({ 
        nome: { $regex: new RegExp(nomeReality, 'i') } 
    });
    
    if (!show) {
        return res.status(404).json({
            sucesso: false,
            mensagem: `Reality show "${nomeReality}" não encontrado`,
            sugestao: 'Verifique o nome ou use GET /reality-shows para listar todos'
        });
    }
    
    if (!show.participantes || show.participantes.length === 0) {
        return res.status(404).json({
            sucesso: false,
            mensagem: 'Nenhum participante encontrado neste reality show'
        });
    }
    
    const participantes = show.participantes;
    const maisNovo = participantes.reduce((prev, curr) => prev.idade < curr.idade ? prev : curr);
    const maisVelho = participantes.reduce((prev, curr) => prev.idade > curr.idade ? prev : curr);
    
    const idades = participantes.map(p => p.idade);
    const idadeMedia = (idades.reduce((sum, idade) => sum + idade, 0) / idades.length).toFixed(1);
    
    res.json({
        sucesso: true,
        reality_show: show.nome,
        emissora: show.emissora.nome,
        estatisticas: {
            total_participantes: participantes.length,
            idade_media: parseFloat(idadeMedia),
            amplitude_idade: maisVelho.idade - maisNovo.idade
        },
        mais_novo: {
            nome: maisNovo.nome,
            idade: maisNovo.idade,
            premios_recebidos: (maisNovo.premios || []).length,
            valor_total_ganho: (maisNovo.premios || []).reduce((sum, p) => sum + p.valor, 0)
        },
        mais_velho: {
            nome: maisVelho.nome,
            idade: maisVelho.idade,
            premios_recebidos: (maisVelho.premios || []).length,
            valor_total_ganho: (maisVelho.premios || []).reduce((sum, p) => sum + p.valor, 0)
        },
        distribuicao_etaria: {
            '20-25 anos': participantes.filter(p => p.idade >= 20 && p.idade <= 25).length,
            '26-30 anos': participantes.filter(p => p.idade >= 26 && p.idade <= 30).length,
            '31-35 anos': participantes.filter(p => p.idade >= 31 && p.idade <= 35).length,
            'acima de 35': participantes.filter(p => p.idade > 35).length
        }
    });
}));

// GET /maior/:valor - Prêmios acima do valor com análise detalhada
app.get('/maior/:valor', asyncHandler(async (req, res) => {
    const valor = parseFloat(req.params.valor);
    
    if (isNaN(valor) || valor < 0) {
        return res.status(400).json({
            sucesso: false,
            mensagem: 'Valor inválido. Informe um número positivo',
            exemplo: '/maior/50000'
        });
    }
    
    const shows = await db.collection('shows').find({}).toArray();
    
    const premiosEncontrados = [];
    
    shows.forEach(show => {
        show.participantes.forEach(participante => {
            (participante.premios || []).forEach(premio => {
                if (premio.valor >= valor) {
                    premiosEncontrados.push({
                        emissora: show.emissora.nome,
                        reality_show: show.nome,
                        participante: participante.nome,
                        idade_participante: participante.idade,
                        premio: {
                            descricao: premio.descricao,
                            valor: premio.valor,
                            data_recebimento: premio.data_recebimento,
                            percentual_acima_filtro: (((premio.valor - valor) / valor) * 100).toFixed(2) + '%'
                        }
                    });
                }
            });
        });
    });
    
    if (premiosEncontrados.length === 0) {
        return res.json({
            sucesso: true,
            mensagem: `Nenhum prêmio encontrado com valor >= R$ ${valor.toLocaleString('pt-BR')}`,
            dados: []
        });
    }
    
    premiosEncontrados.sort((a, b) => b.premio.valor - a.premio.valor);
    
    const estatisticas = {
        total_premios_encontrados: premiosEncontrados.length,
        valor_minimo_filtro: valor,
        maior_premio: premiosEncontrados[0].premio.valor,
        menor_premio: premiosEncontrados[premiosEncontrados.length - 1].premio.valor,
        soma_total: premiosEncontrados.reduce((sum, p) => sum + p.premio.valor, 0),
        valor_medio: parseFloat((premiosEncontrados.reduce((sum, p) => sum + p.premio.valor, 0) / premiosEncontrados.length).toFixed(2)),
        emissoras_envolvidas: [...new Set(premiosEncontrados.map(p => p.emissora))],
        reality_shows_envolvidos: [...new Set(premiosEncontrados.map(p => p.reality_show))]
    };
    
    res.json({
        sucesso: true,
        filtro_aplicado: `Prêmios >= R$ ${valor.toLocaleString('pt-BR')}`,
        estatisticas,
        premios: premiosEncontrados
    });
}));

// GET /total - VERSÃO CORRIGIDA E SIMPLIFICADA
app.get('/total', asyncHandler(async (req, res) => {
    try {
        const shows = await db.collection('shows').find({}).toArray();
        
        if (!shows || shows.length === 0) {
            return res.status(404).json({
                sucesso: false,
                mensagem: 'Nenhum reality show encontrado'
            });
        }
        
        const resultado = [];
        let totalGeralValor = 0;
        let totalGeralPremios = 0;
        let maiorPremioGlobal = 0;
        
        shows.forEach(show => {
            const todosPremios = [];
            const participantesComPremio = new Set();
            
            // Coletar todos os prêmios do reality show
            show.participantes.forEach(participante => {
                if (participante.premios && participante.premios.length > 0) {
                    participante.premios.forEach(premio => {
                        todosPremios.push({
                            participante: participante.nome,
                            descricao: premio.descricao,
                            valor: premio.valor,
                            data: premio.data_recebimento
                        });
                        participantesComPremio.add(participante.nome);
                    });
                }
            });
            
            // Calcular estatísticas
            const totalValor = todosPremios.reduce((sum, p) => sum + p.valor, 0);
            const quantidadePremios = todosPremios.length;
            
            totalGeralValor += totalValor;
            totalGeralPremios += quantidadePremios;
            
            const maiorPremio = quantidadePremios > 0 ? Math.max(...todosPremios.map(p => p.valor)) : 0;
            const menorPremio = quantidadePremios > 0 ? Math.min(...todosPremios.map(p => p.valor)) : 0;
            const premioMedio = quantidadePremios > 0 ? totalValor / quantidadePremios : 0;
            
            if (maiorPremio > maiorPremioGlobal) {
                maiorPremioGlobal = maiorPremio;
            }
            
            // Top 3 maiores prêmios
            const top3 = [...todosPremios]
                .sort((a, b) => b.valor - a.valor)
                .slice(0, 3);
            
            resultado.push({
                reality_show: show.nome,
                emissora: show.emissora.nome,
                total_distribuido: totalValor,
                quantidade_premios: quantidadePremios,
                participantes_premiados: participantesComPremio.size,
                total_participantes: show.participantes.length,
                maior_premio: maiorPremio,
                menor_premio: menorPremio,
                premio_medio: parseFloat(premioMedio.toFixed(2)),
                percentual_participantes_premiados: show.participantes.length > 0 
                    ? ((participantesComPremio.size / show.participantes.length) * 100).toFixed(2) + '%'
                    : '0%',
                top_3_maiores_premios: top3
            });
        });
        
        // Ordenar por total distribuído (decrescente)
        resultado.sort((a, b) => b.total_distribuido - a.total_distribuido);
        
        const resumoGeral = {
            total_reality_shows: resultado.length,
            valor_total_distribuido: totalGeralValor,
            total_premios_distribuidos: totalGeralPremios,
            media_premios_por_reality: parseFloat((totalGeralPremios / resultado.length).toFixed(2)),
            media_valor_por_reality: parseFloat((totalGeralValor / resultado.length).toFixed(2)),
            reality_mais_generoso: resultado[0]?.reality_show || 'N/A',
            maior_premio_individual: maiorPremioGlobal
        };
        
        res.json({
            sucesso: true,
            resumo: resumoGeral,
            detalhamento_por_reality: resultado
        });
        
    } catch (error) {
        console.error('Erro no /total:', error);
        res.status(500).json({
            sucesso: false,
            mensagem: 'Erro ao processar dados',
            erro: error.message
        });
    }
}));

// GET /audiencia - Análise de audiência
app.get('/audiencia', asyncHandler(async (req, res) => {
    const shows = await db.collection('shows').find({}).toArray();
    
    const emissoras = {};
    
    shows.forEach(show => {
        const nomeEmissora = show.emissora.nome;
        
        if (!emissoras[nomeEmissora]) {
            emissoras[nomeEmissora] = {
                emissora: nomeEmissora,
                pontos_audiencia: show.emissora.pontos_audiencia,
                reality_shows: [],
                total_participantes: 0,
                total_premios_distribuidos: 0
            };
        }
        
        emissoras[nomeEmissora].reality_shows.push(show.nome);
        emissoras[nomeEmissora].total_participantes += show.participantes.length;
        
        show.participantes.forEach(p => {
            (p.premios || []).forEach(premio => {
                emissoras[nomeEmissora].total_premios_distribuidos += premio.valor;
            });
        });
    });
    
    const resultado = Object.values(emissoras).map(e => ({
        ...e,
        quantidade_reality_shows: e.reality_shows.length,
        investimento_por_ponto: e.pontos_audiencia > 0 ? 
            parseFloat((e.total_premios_distribuidos / e.pontos_audiencia).toFixed(2)) : 0
    })).sort((a, b) => b.pontos_audiencia - a.pontos_audiencia);
    
    const totalPontos = resultado.reduce((sum, r) => sum + r.pontos_audiencia, 0);
    const totalInvestimento = resultado.reduce((sum, r) => sum + r.total_premios_distribuidos, 0);
    
    const resultadoFinal = resultado.map((r, index) => ({
        posicao: index + 1,
        emissora: r.emissora,
        pontos_audiencia: r.pontos_audiencia,
        participacao_mercado: ((r.pontos_audiencia / totalPontos) * 100).toFixed(2) + '%',
        reality_shows: {
            quantidade: r.quantidade_reality_shows,
            nomes: r.reality_shows
        },
        total_participantes: r.total_participantes,
        total_premios_distribuidos: r.total_premios_distribuidos,
        investimento_por_ponto_audiencia: r.investimento_por_ponto,
        eficiencia: r.pontos_audiencia > 70 ? 'Alta' : r.pontos_audiencia > 60 ? 'Média' : 'Baixa'
    }));
    
    res.json({
        sucesso: true,
        resumo: {
            total_emissoras: resultado.length,
            total_pontos_audiencia: totalPontos,
            total_investido_premios: totalInvestimento,
            media_pontos_por_emissora: parseFloat((totalPontos / resultado.length).toFixed(2)),
            lider_audiencia: resultado[0]?.emissora || 'N/A'
        },
        ranking_emissoras: resultadoFinal
    });
}));

// GET /reality-shows - Listar todos
app.get('/reality-shows', asyncHandler(async (req, res) => {
    const shows = await db.collection('shows').find({}).toArray();
    
    res.json({
        sucesso: true,
        total: shows.length,
        reality_shows: shows.map(s => ({
            nome: s.nome,
            emissora: s.emissora.nome,
            total_participantes: s.participantes.length
        }))
    });
}));

// GET / - Documentação
app.get('/', (req, res) => {
    res.json({
        api: 'Reality Show API',
        versao: '2.0',
        endpoints: [
            { metodo: 'GET', rota: '/premios', descricao: 'Lista todos os reality shows com participantes e prêmios' },
            { metodo: 'GET', rota: '/idade/:nome_reality', descricao: 'Participante mais novo e mais velho' },
            { metodo: 'GET', rota: '/maior/:valor', descricao: 'Prêmios >= valor informado' },
            { metodo: 'GET', rota: '/total', descricao: 'Total de prêmios por reality show' },
            { metodo: 'GET', rota: '/audiencia', descricao: 'Ranking de audiência das emissoras' },
            { metodo: 'GET', rota: '/reality-shows', descricao: 'Lista todos os reality shows' }
        ]
    });
});

// Tratamento de erros
app.use((err, req, res, next) => {
    console.error('Erro:', err);
    res.status(500).json({
        sucesso: false,
        mensagem: 'Erro interno do servidor',
        erro: err.message
    });
});

// 404
app.use((req, res) => {
    res.status(404).json({
        sucesso: false,
        mensagem: 'Endpoint não encontrado',
        sugestao: 'Acesse GET / para documentação'
    });
});

// Iniciar
const start = async () => {
    try {
        const client = new MongoClient(url, { useUnifiedTopology: true });
        await client.connect();
        console.log('✓ Conectado ao MongoDB');
        
        db = client.db('reality');
        
        app.listen(3000, () => {
            console.log('\n🚀 Servidor rodando na porta 3000\n');
            console.log('📚 Documentação: http://localhost:3000/\n');
        });
    } catch (err) {
        console.error('❌ Erro ao conectar:', err.message);
        process.exit(1);
    }
};

start();
EOF
```

#### 8. **Iniciar o Servidor:**

```bash
node server.js
```

Você deve ver:
```
✓ Conectado ao MongoDB
🚀 Servidor rodando na porta 3000
📚 Documentação: http://localhost:3000/
```

---

## 🎯 Acessando a Aplicação

No **Play with Docker**, clique no link da porta **3000** que aparece no topo da sua instância.

---

## 🧪 Testando a API

### Opção 1: Via Browser
Acesse diretamente pelo navegador:
- `http://ip172-18-0-X-XXXX.direct.labs.play-with-docker.com:3000/`
- `http://ip172-18-0-X-XXXX.direct.labs.play-with-docker.com:3000/reality-shows`

### Opção 2: Via cURL (em uma nova instância)

```bash
# Adicionar nova instância e instalar curl
apk add curl

# Definir IP do servidor (substitua X pelo IP da primeira instância)
export SERVER=192.168.0.X

# 1. Ver documentação
curl http://$SERVER:3000/

# 2. Listar reality shows
curl http://$SERVER:3000/reality-shows

# 3. Ver todos os prêmios
curl http://$SERVER:3000/premios

# 4. Participante mais novo/velho (Big Brother)
curl "http://$SERVER:3000/idade/Big%20Brother%20Brasil%202025"

# 5. Prêmios acima de R$ 50.000
curl http://$SERVER:3000/maior/50000

# 6. Total por reality show
curl http://$SERVER:3000/total

# 7. Ranking de audiência
curl http://$SERVER:3000/audiencia
```

---

## 📁 Estrutura do Projeto

```
reality-app/
├── package.json
├── server.js
└── reality_shows.json
```

---

## 📊 Modelagem NoSQL

### Documento Reality Show:
```javascript
{
  "_id": ObjectId(),
  "nome": "Big Brother Brasil 2025",
  "emissora": {
    "nome": "Globo",
    "pontos_audiencia": 85
  },
  "participantes": [
    {
      "nome": "João Silva",
      "idade": 28,
      "premios": [
        {
          "descricao": "Carro 0km",
          "valor": 80000,
          "data_recebimento": "2025-03-15"
        }
      ]
    }
  ]
}
```

**Vantagens do modelo:**
- ✅ Documentos aninhados (embedded documents)
- ✅ Sem necessidade de JOINs
- ✅ Dados relacionados ficam juntos
- ✅ Performance otimizada para leituras

---

## 📝 Documentação da API

| Método | Endpoint | Descrição | Exemplo |
|--------|----------|-----------|---------|
| GET | `/` | Documentação da API | `/` |
| GET | `/reality-shows` | Lista todos os reality shows | `/reality-shows` |
| GET | `/premios` | Todos os prêmios distribuídos com estatísticas | `/premios` |
| GET | `/idade/:nome` | Participante mais novo e mais velho | `/idade/Big%20Brother` |
| GET | `/maior/:valor` | Prêmios maiores ou iguais ao valor | `/maior/50000` |
| GET | `/total` | Total de prêmios por reality show | `/total` |
| GET | `/audiencia` | Ranking de audiência das emissoras | `/audiencia` |

### Exemplos de Respostas:

#### GET `/reality-shows`
```json
{
  "sucesso": true,
  "total": 3,
  "reality_shows": [
    {
      "nome": "Big Brother Brasil 2025",
      "emissora": "Globo",
      "total_participantes": 10
    }
  ]
}
```

#### GET `/idade/Big Brother Brasil 2025`
```json
{
  "sucesso": true,
  "reality_show": "Big Brother Brasil 2025",
  "estatisticas": {
    "total_participantes": 10,
    "idade_media": 28.5,
    "amplitude_idade": 9
  },
  "mais_novo": {
    "nome": "Isabela Dias",
    "idade": 24
  },
  "mais_velho": {
    "nome": "João Oliveira",
    "idade": 33
  }
}
```

---

## 🔧 Comandos Úteis

### Verificar status do MongoDB:
```bash
ps aux | grep mongod
```

### Ver dados no MongoDB:
```bash
mongo
use reality
db.shows.find().pretty()
db.shows.count()
exit
```

### Reiniciar o servidor:
```bash
# Pressione Ctrl+C para parar
node server.js
```

### Ver logs em tempo real:
```bash
# Execute em outra janela/aba
tail -f /var/log/mongodb.log
```

---

## 🐛 Troubleshooting

### ❌ Erro: "Cannot connect to MongoDB"
**Solução:** Verifique se o MongoDB está rodando:
```bash
ps aux | grep mongod
# Se não estiver, inicie:
mongod --dbpath /root/mongodb --fork --logpath /dev/null --bind_ip_all
```

### ❌ Erro: "Collection 'shows' not found"
**Solução:** Reimporte os dados:
```bash
cd ~/reality-app
mongoimport --db reality --collection shows --file reality_shows.json --jsonArray
```

### ❌ Erro: "Cannot GET /endpoint"
**Solução:** Verifique se o servidor está rodando:
```bash
ps aux | grep node
# Se não estiver:
cd ~/reality-app
node server.js
```

### ❌ Porta 3000 já está em uso
**Solução:** Mate o processo:
```bash
pkill -f "node server.js"
node server.js
```

---

## 🎓 Conceitos Aprendidos

✅ **MongoDB** - Banco NoSQL orientado a documentos  
✅ **Express.js** - Framework web para Node.js  
✅ **REST API** - Arquitetura de APIs RESTful  
✅ **Embedded Documents** - Documentos aninhados  
✅ **Aggregation Pipeline** - Agregações no MongoDB  
✅ **Error Handling** - Tratamento de erros assíncrono  

---

## 📚 Recursos Adicionais

- [Documentação MongoDB](https://docs.mongodb.com/)
- [Express.js Guide](https://expressjs.com/pt-br/)
- [MongoDB Aggregation](https://docs.mongodb.com/manual/aggregation/)
- [REST API Best Practices](https://restfulapi.net/)

---

## 📄 Licença

Este projeto é livre para uso educacional.

---

**Desenvolvido para aprendizado de NoSQL e APIs RESTful** 🚀
