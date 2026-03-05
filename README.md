🌤️ Telegram Weather Chatbot (n8n)

Um chatbot automatizado no Telegram que retorna a temperatura atual de uma cidade brasileira utilizando a API OpenWeather e automação com n8n.

O bot recebe uma mensagem no formato Cidade,UF e responde com a temperatura atual em graus Celsius.

📌 Demonstração

Entrada do usuário
Uberlândia,MG
Resposta do bot
🌤️ A temperatura em Uberlândia é de 20°C.

Caso a cidade não seja encontrada:
❌ Cidade não encontrada. Use o formato Cidade,UF (ex.: São Paulo,SP).

🚀 Tecnologias utilizadas

n8n — Automação de workflows

Telegram Bot API — Comunicação com o usuário

OpenWeather API — Consulta de dados meteorológicos

JavaScript Expressions (n8n) — Processamento de dados

⚙️ Arquitetura do Workflow

O fluxo de automação é composto pelos seguintes nós:

Telegram Trigger
      ↓
Edit Fields (queue)
      ↓
HTTP Request (OpenWeather API)
      ↓
IF (validação da resposta)
   ↓        ↓
True      False
 ↓          ↓
Format     Telegram Error Message
Message
 ↓
Telegram Send Message
🔄 Funcionamento do Bot
1️⃣ Telegram Trigger

O workflow inicia quando o bot recebe uma mensagem do usuário no Telegram.

Formato esperado da mensagem:

Cidade,UF

Exemplo:
Uberlândia,MG

2️⃣ Normalização da entrada

A mensagem enviada é transformada em uma variável chamada queue, que será utilizada na consulta da API.

Expressão utilizada no n8n:

{{
(() => {
  const raw = ($json.message?.text || '').trim();

  let s = raw.normalize('NFD').replace(/[\u0300-\u036f]/g, '').toLowerCase();

  s = s.replace(/\s+/g, ' ').trim();

  if (!s.includes(',')) {
    const parts = s.split(' ');
    if (parts.length >= 2) {
      const uf = parts.pop().toUpperCase();
      const city = parts.join(' ');
      s = `${city},${uf}`;
    }
  } else {
    const [city, uf] = s.split(',').map(t => t.trim());
    s = `${city},${(uf || '').toUpperCase()}`;
  }

  if (!s.endsWith(',BR')) s = `${s},BR`;

  return s;
})()
}}

Exemplo de transformação:

Entrada do usuário:
Uberlândia,MG

queue gerado:
Uberlândia,MG,BR

Adicionar ",BR" melhora a precisão da busca na API OpenWeather.

3️⃣ Consulta à OpenWeather API

O node HTTP Request realiza a consulta:

https://api.openweathermap.org/data/2.5/weather
Parâmetros utilizados
Parâmetro	Descrição
q	cidade formatada
units	metric (graus Celsius)
lang	pt_br
appid	chave da API

Exemplo de requisição:
https://api.openweathermap.org/data/2.5/weather?q=Uberlândia,MG,BR

4️⃣ Validação da resposta

Um node IF verifica se a resposta da API é válida.

Condições avaliadas:

statusCode = 200
AND
body.main.temp não está vazio

Se a resposta for válida, o fluxo continua para formatação da mensagem.

Caso contrário, o bot retorna uma mensagem de erro.

5️⃣ Formatação da resposta

A temperatura é extraída da resposta da API e formatada.

Expressão utilizada:

{{`🌤️ A temperatura em ${$json.body.name} é de ${Math.round($json.body.main.temp)}°C.`}}

Exemplo de saída:
🌤️ A temperatura em Uberlândia é de 20°C.

6️⃣ Envio da resposta no Telegram

O node Telegram Send Message envia a resposta ao usuário utilizando:

chat.id

Isso garante que o bot responda diretamente na conversa do usuário.

❌ Tratamento de erros

Quando a cidade não é encontrada ou ocorre erro na requisição, o bot responde:

❌ Cidade não encontrada. Use o formato Cidade,UF (ex.: São Paulo,SP).
🔐 Variáveis de ambiente

Para executar o projeto é necessário configurar as seguintes variáveis:

OpenWeather API
OPENWEATHER_API_KEY

Obtenha gratuitamente em:

https://openweathermap.org/api

Telegram Bot Token
TELEGRAM_BOT_TOKEN

Crie um bot utilizando o BotFather no Telegram.

📦 Como executar o projeto

1️⃣ Instalar o n8n
npm install n8n -g

Ou utilize n8n Cloud.

2️⃣ Importar o workflow

No painel do n8n:

Import → workflow-telegram-chatbot.json
3️⃣ Configurar credenciais

Adicione:

Telegram Bot Token

OpenWeather API Key

4️⃣ Ativar o workflow

Clique em Activate no n8n.

5️⃣ Testar o bot

Envie uma mensagem no Telegram:

Uberlândia,MG

Resposta esperada:

🌤️ A temperatura em Uberlândia é de XX°C.

📁 Estrutura do projeto

telegram-weather-chatbot
│
├── workflow-telegram-chatbot.json
├── README.md
├── README.txt
└── .env.example

📜 Licença
Projeto desenvolvido para fins educacionais no desafio Rocketseat utilizando n8n.

