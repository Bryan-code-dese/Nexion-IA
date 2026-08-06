// api/chat.js
// Função serverless (formato Vercel) que fala com a Claude por trás do site.
// O visitante do seu site NUNCA vê nem precisa de nenhuma chave ou conta.
// A chave fica guardada apenas aqui no servidor, como variável de ambiente.

export default async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).json({ reply: 'Método não permitido.' });
  }

  const { message } = req.body || {};
  if (!message || typeof message !== 'string') {
    return res.status(400).json({ reply: 'Mensagem inválida.' });
  }

  try {
    const response = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': process.env.ANTHROPIC_API_KEY, // configurada no painel da Vercel, nunca no código
        'anthropic-version': '2023-06-01'
      },
      body: JSON.stringify({
        model: 'claude-sonnet-4-6',
        max_tokens: 1000,
        system: "Você é o Nexion IA, um assistente de negócios direto e prático que ajuda iniciantes a conseguir sua primeira venda. Responda em português do Brasil, de forma objetiva, com passos concretos e acionáveis. Quando fizer sentido, ofereça estrutura de site, ideia de produto, estratégia de lançamento ou plano de venda. Seja encorajador mas realista, sem promessas exageradas.",
        messages: [{ role: 'user', content: message }]
      })
    });

    const data = await response.json();
    const reply = (data.content || [])
      .filter((b) => b.type === 'text')
      .map((b) => b.text)
      .join('\n') || 'Não consegui gerar uma resposta agora, tenta reformular a pergunta.';

    return res.status(200).json({ reply });
  } catch (err) {
    console.error('Erro ao chamar a API da Claude:', err);
    return res.status(500).json({ reply: 'Deu um erro ao falar com o Nexion agora. Tenta de novo em instantes.' });
  }
}
