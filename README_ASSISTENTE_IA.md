# Assistente virtual da Vida de Cão

Este pacote mantém o site existente e acrescenta um chat compacto no canto inferior esquerdo. O navegador envia as perguntas para `/api/chat`; a função no servidor consulta a OpenAI Responses API usando uma chave guardada fora dos arquivos públicos. A base de fatos editável está em `data/clinic-info.js`.

**Importante:** GitHub Pages não executa a função de servidor. O repositório pode continuar no GitHub, mas conecte este projeto à Vercel para servir `api/chat.js`; publicar apenas os arquivos estáticos no GitHub Pages deixa a IA indisponível.

## O que o assistente pode responder

O assistente usa somente informações encontradas no HTML atual: endereço, atendimento 24 horas, telefones, WhatsApp, serviços, estrutura e Instagram. O servidor identifica perguntas sobre preços/pagamentos e resultados/laudos e responde com uma mensagem fixa, sem enviá-las ao modelo. Relatos comuns de sintomas e termos de emergência também são encaminhados sem chamar o modelo: a pessoa recebe um link para abrir o WhatsApp com um rascunho contendo o sintoma e outro para ligar à clínica. O texto não é enviado automaticamente; a pessoa pode revisar e decidir se envia. O assistente não avalia sintomas, não diagnostica e não prescreve. Ele não tem acesso a prontuários ou sistemas de pacientes. Valores, disponibilidade, pagamentos e horários específicos do Pet Táxi não foram informados; por isso, o chat orienta a confirmar com a equipe.

O texto das respostas é inserido como texto simples no navegador, sem executar HTML recebido da IA. O histórico vive apenas na memória da página aberta; não há banco de conversas nem `localStorage`, então não há atualmente uma tela/local para consultar transcrições. Ao fechar ou recarregar a página, a conversa do navegador se perde. Nas perguntas encaminhadas ao modelo, a função recebe até 10 mensagens recentes e a base pública da clínica, e envia esses dados à OpenAI. Emergências, sintomas identificados, preços, resultados de exames e pedidos fora do escopo recebem tratamento fixo no servidor e não são enviados ao modelo. A requisição ao modelo usa `store: false`; isso desativa o armazenamento da resposta para recuperação pela API, mas não deve ser interpretado como promessa de retenção zero por todos os serviços envolvidos. A infraestrutura da hospedagem também processa os pedidos. Para manter histórico consultável seria necessário implementar armazenamento próprio com aviso/consentimento, acesso restrito, prazo de retenção e cuidado adicional por envolver dados de saúde de animais e possivelmente dados pessoais.

## Testar localmente no Windows

1. Extraia o ZIP. A pasta extraída deve conter `index.html`, `equipe.html`, `assets`, `api`, `data` e `package.json` no mesmo nível.
2. Instale o Node.js 20 ou mais recente, se ainda não estiver instalado.
3. Abra o Prompt de Comando nessa pasta e rode:

   ```cmd
   npm test
   npm run dev
   ```

4. Na primeira execução, o Vercel CLI pode pedir para entrar na sua conta. Abra o endereço local indicado no terminal. O `file://` não executa a função `/api/chat`; use o endereço servido por `vercel dev`.
5. Para uma conversa real local, copie `.env.example` para `.env.local`, abra esse arquivo no Bloco de Notas e preencha `OPENAI_API_KEY` com sua chave. O `.env.local` está excluído do Git. Reinicie `npm run dev`. Não cole a chave neste README, no HTML ou em mensagens.

Sem uma chave válida, o chat ainda abre e mantém o WhatsApp disponível, mas a resposta da IA mostra uma mensagem de indisponibilidade.

## Configurar na Vercel

1. Importe ou conecte este projeto na Vercel. A raiz do projeto precisa ser a pasta que contém `index.html` e `api/chat.js`.
2. Em **Settings → Environment Variables**, crie `OPENAI_API_KEY` com a chave da API da OpenAI nos ambientes necessários. Nunca crie uma variável `VITE_` ou `NEXT_PUBLIC_` para essa chave.
3. Opcionalmente configure `OPENAI_MODEL`. O padrão é `gpt-6-luna`, um modelo eficiente para tarefas de texto focadas e alto volume. Pode ser substituído por outro modelo disponível à conta e compatível com a Responses API. Consulte a página de [modelos](https://developers.openai.com/api/docs/models) e [preços](https://developers.openai.com/api/docs/pricing) da OpenAI antes de escolher; o gasto varia com o volume e os tokens enviados e gerados.
4. Faça um novo deploy depois de adicionar ou alterar variáveis. A Vercel só aplica alterações de variáveis aos novos deploys.
5. Em **Firewall → Configure → New Rule**, limite pedidos `POST` para o caminho `/api/chat` por IP (por exemplo, 8 pedidos por minuto). O código também tem um limite temporário em memória por instância, útil como proteção básica; em funções serverless, reinicializações e múltiplas instâncias podem reiniciar ou repartir esse contador. A regra de Firewall é a proteção compartilhada recomendada para produção.
6. Na conta da OpenAI, acompanhe o uso e configure alertas/limites de gasto. Um limite rígido pode interromper respostas quando for atingido.

## GitHub sem publicar segredos

Inclua o HTML, os assets, `api/chat.js`, `data/clinic-info.js`, `package.json`, `vercel.json`, `.gitignore` e este README no repositório. O `.gitignore` exclui `.env`, `.env.local` e arquivos locais da Vercel. Antes de enviar, confira que nenhuma chave real aparece no histórico ou nos arquivos. Se uma chave for publicada por engano, revogue-a e crie outra.

## Atualizar as informações

Edite `data/clinic-info.js` apenas com dados confirmados pela clínica. As mesmas informações são fornecidas às instruções da IA no servidor. Não acrescente preços, disponibilidade, formas de pagamento ou nomes/especialidades novos sem confirmação. Telefones, horários ou links alterados devem ser revisados também no HTML da página.

## O que foi testado

- `npm test`: testes locais simulam as respostas da OpenAI; cobrem método inválido, corpo malformado/grande, emergência e sintomas encaminhados sem chamar a IA, links do WhatsApp com rascunho e da ligação, roteamento de preço/resultado de exame e recusa de pedidos fora do escopo, resposta bem-sucedida, falta de chave e limite básico de pedidos. Eles não consomem API nem provam que sua chave ou conta têm acesso ao modelo.
- A interface deve ser conferida via `npm run dev` em desktop e celular. O envio real depende de chave válida configurada na Vercel/local e de uma implantação da função.
- Não foi feito deploy nem chamada real à API. A chave não foi solicitada nem incluída nos arquivos.

## Referências oficiais consultadas

- [OpenAI Responses API — geração de texto](https://developers.openai.com/api/docs/guides/text)
- [OpenAI Responses API — referência de criação](https://developers.openai.com/api/reference/cli/resources/responses/methods/create)
- [OpenAI modelos](https://developers.openai.com/api/docs/models)
- [OpenAI preços](https://developers.openai.com/api/docs/pricing)
- [OpenAI boas práticas de produção e proteção de chaves](https://developers.openai.com/api/docs/guides/production-best-practices)
- [Vercel Functions para Node.js](https://vercel.com/docs/functions/runtimes/node-js)
- [Variáveis de ambiente da Vercel](https://vercel.com/docs/environment-variables)
- [Rate limiting do Vercel Firewall](https://vercel.com/docs/vercel-firewall/vercel-waf/rate-limiting)

