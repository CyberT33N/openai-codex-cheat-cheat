Es besteht die Möglichkeit, über die Config-Trommel per Property Custom Instructions zu setzen.

Zusätzlich gibt es noch eine Agents MD, und im Einstellungsbereich hat man ebenfalls die Möglichkeit, Custom Instructions zu hinterlegen. An sich funktioniert das auch mit dieser Soul MD.

Das Problem ist, dass OpenAI System Prompts limitiert, weil sie über ein Agent Framework laufen. Das heißt, wirklich system-prompt-spezifische Anforderungen, also etwa wie Tools aufgerufen werden und so weiter, werden über die höher gelagerte Orchestrierungsebene von OpenAI blockiert. Das ist genau das Gleiche wie bei Cursor und anderen IDEs.

Daher ist es nicht möglich, extrem spezifische System Prompts zu gestalten, sondern nur allgemeine Anforderungen.

```
model_instructions_file = 'soul.md'
```
