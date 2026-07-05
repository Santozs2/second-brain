---
type: technology
type: tech
tags:
  - tech
  - estudo
  - ai
  - voz
tecnologia: Assistente de Voz
status: aprendendo
created: 2026-07-02
updated: 2026-07-02
---

# Assistente de Voz (Wake Word → STT → LLM → TTS)

> [!tip] Status
> 🟡 Aprendendo

## 📝 Resumo

Pipeline típico de um assistente de voz "tipo Alexa/Siri": um trecho de áudio percorre várias etapas até virar uma resposta falada. O fluxo básico é:

```text
🎤 Áudio
  ↓
Wake Word (ativação)
  ↓
STT (fala → texto)
  ↓
Orquestrador / LLM (entende e decide)
  ↓
Skill Dispatcher (roteia para a skill certa)
  ↓
Skill (executa a ação)
  ↓
TTS (texto → fala)
  ↓
🔊 Resposta falada
```

Cada etapa é um componente separado e substituível — o que permite trocar de provedor (ex: outro STT, outro LLM) sem reescrever o pipeline inteiro.

## 🧠 Conceitos principais

- **Wake Word (palavra de ativação)**
  - Modelo pequeno e sempre rodando localmente (ex: "Alexa", "Ok Google", "Hey Siri") que escuta continuamente em busca de um padrão sonoro específico.
  - Precisa ser leve (roda no dispositivo, não na nuvem) e ter baixo consumo de energia/CPU, já que fica ativo o tempo todo.
  - Só depois de detectar a wake word é que o áudio seguinte é enviado para as próximas etapas (evita gravar/enviar tudo o tempo todo — privacidade e custo).
  - Exemplos de ferramentas: Porcupine (Picovoice), Snowboy (descontinuado), openWakeWord.

- **STT — Speech-to-Text (fala → texto)**
  - Converte o áudio captado após a wake word em texto transcrito.
  - Pode rodar localmente (Whisper, Vosk) ou via API na nuvem (OpenAI Whisper API, Google Speech-to-Text, Azure Speech, Deepgram).
  - Trade-off: local = mais privado e sem latência de rede, mas exige mais poder de processamento; nuvem = mais preciso, porém depende de internet e tem custo por uso.

- **Orquestrador (LLM)**
  - Recebe o texto transcrito e é responsável por entender a intenção do usuário.
  - Decide se responde diretamente (conversa) ou se precisa acionar uma ação/skill (ex: "que horas são", "toca uma música").
  - É o "cérebro" do pipeline — normalmente um LLM (GPT, Claude, Gemini) com um prompt de sistema definindo seu papel e as skills disponíveis.
  - Pode usar function calling / tool use para decidir estruturadamente qual skill chamar e com quais parâmetros.

- **Skill Dispatcher (roteador de skills)**
  - Componente que recebe a decisão do orquestrador (qual skill + parâmetros) e a direciona para a implementação correta.
  - Funciona como um "switch"/router entre a intenção detectada e o código que efetivamente executa a ação.
  - Cuida de validação de parâmetros, tratamento de erros e fallback (ex: skill não encontrada → resposta padrão).

- **Skills**
  - Módulos independentes que executam uma ação concreta: tocar música, consultar clima, criar lembrete, controlar dispositivo smart home, responder pergunta de conhecimento, etc.
  - Cada skill tem uma interface bem definida (entrada/saída) para poder ser adicionada ou removida sem afetar o resto do sistema.
  - Análogo aos "tools" de um agente de IA moderno.

- **TTS — Text-to-Speech (texto → fala)**
  - Converte a resposta final (texto) em áudio falado.
  - Pode ser local (Piper, Coqui TTS) ou via API (ElevenLabs, Azure TTS, Google TTS, OpenAI TTS).
  - Trade-offs parecidos com o STT: qualidade/naturalidade de voz vs. latência, custo e privacidade.

## 💻 Exemplos

```text
Fluxo de exemplo:

1. Usuário: "Hey Assistente..."
   → Wake word detectada localmente (ex: Porcupine)

2. Usuário: "...que horas são em Tóquio?"
   → Áudio capturado e enviado ao STT
   → STT retorna: "que horas são em Tóquio"

3. Orquestrador (LLM) recebe o texto
   → Identifica intenção: consulta de horário
   → Decide chamar a skill "horario_mundial" com parâmetro cidade="Tóquio"

4. Skill Dispatcher
   → Roteia a chamada para o módulo da skill correspondente

5. Skill "horario_mundial"
   → Executa a lógica (ex: consulta timezone) e retorna: "são 22h47 em Tóquio"

6. Orquestrador formata a resposta final em linguagem natural
   → "Agora são 22h47 em Tóquio."

7. TTS converte o texto em áudio
   → Resposta é reproduzida no alto-falante
```

```json
// Exemplo simplificado de "intenção" estruturada que o LLM devolveria
// para o skill dispatcher decidir o que executar
{
  "intent": "horario_mundial",
  "parameters": {
    "cidade": "Tóquio"
  }
}
```

## 🔗 Links úteis

- [Picovoice Porcupine (wake word)](https://picovoice.ai/platform/porcupine/)
- [OpenAI Whisper (STT)](https://openai.com/research/whisper)
- [OpenAI Text-to-Speech](https://platform.openai.com/docs/guides/text-to-speech)
- [ElevenLabs (TTS)](https://elevenlabs.io/)

## ✅ Checklist de aprendizado

- [ ] Rodar um wake word engine localmente (ex: Porcupine ou openWakeWord)
- [ ] Testar STT local (Whisper) vs. STT via API
- [ ] Montar um orquestrador simples com function calling / tool use
- [ ] Implementar um skill dispatcher básico (roteamento por intent)
- [ ] Criar 2-3 skills de exemplo (hora, clima, lembrete)
- [ ] Testar TTS local vs. TTS via API
- [ ] Integrar o pipeline completo ponta a ponta

## 🗒️ Notas pessoais


## 🧩 Conceitos relacionados

- [[Metas/AMEA AI|AMEA AI]] — ideia futura de "voz em tempo real" no roadmap
- [[Conceitos/Conceitos|Conceitos]]

## 🔗 Veja também

- [[Estudos/Python|Python]]

