# Diseño de solución extremo a extremo para sistema de préstamos

El sistema de préstamos de una entidad financiera requiere una solución que integre los procesos de solicitud, evaluación, aprobación y desembolso de préstamos. El sistema debe soportar un throughput de 10 000 solicitudes por hora con un SLA de 99.9%. Los datos de los clientes son suministrados por un buró de crédito externo, el cual puede experimentar latencias de hasta 2 segundos en respuestas. La solución debe asegurar la idempotencia de las solicitudes de préstamo por número de operación y canal, evitando duplicados en caso de reintentos dentro de un periodo de 24 horas. Además, debe emitir un evento al sistema de auditoría por cada aceptación de préstamo.

## Informacion General

| Campo | Valor |
|-------|-------|
| **Tema** | Diseno de solucion extremo a extremo 1790190135969 |
| **Nivel** | senior-l2 |
| **Tipo** | mixed |
| **Tiempo estimado** | 40 horas |

## Fases del Reto

### Fase 0: Configuración del Proyecto

**Objetivo:** Obtener el proyecto base funcional enviando el Código Base a un asistente de IA, que lo analizará, corregirá errores y generará un ZIP listo para usar.

**Tiempo estimado:** 15-30 minutos

**Instrucciones:**

- Asegúrate de tener instalado para ejecutar el proyecto: Un IDE o editor de código.
- Copia todo el contenido del campo **Código Base** de este reto — incluyendo el texto de instrucciones que aparece al inicio.
- Abre un asistente de IA (Claude en claude.ai, ChatGPT o Gemini — se recomienda Claude), pega el contenido copiado en el chat y envíalo.
- El asistente analizará los archivos, corregirá errores y generará un archivo ZIP descargable. Descárgalo y extráelo en la carpeta donde quieras trabajar.
- Verifica que el proyecto arranca sin errores.

**Entregable:** El proyecto compila/arranca sin errores.

<details>
<summary>Pistas de conocimiento</summary>

- Copia el Código Base completo incluyendo el texto de instrucciones al inicio — esas instrucciones le indican al asistente exactamente qué hacer con los archivos.
- Si el asistente no genera el ZIP automáticamente al terminar el análisis, escríbele: "genera el ZIP ahora".
- Si el proyecto tiene errores al arrancar, comparte el mensaje de error con el mismo asistente para que lo corrija.

</details>

### Fase 1: Especificación del dominio y requisitos

**Objetivo:** Definir claramente los actores, fuentes, sumideros y reglas de negocio para el sistema de préstamos.

**Tiempo estimado:** 10 horas

**Instrucciones:**

- Identificar y describir los actores involucrados en el proceso de préstamos (ej. originador de créditos, motor antifraude, buró de riesgos, core bancario, gateway de pagos, sistema de liquidación, agente de retención, consolidador contable).
- Enumerar las fuentes de datos y sumideros relevantes para el sistema.
- Especificar las reglas de negocio y umbrales numéricos (ej. throughput de 10 000 solicitudes por hora, latencia máxima de 2 segundos en respuestas del buró).
- Definir la idempotencia de las solicitudes de préstamo por número de operación y canal, y el evento de auditoría por cada aceptación.

**Entregable:** Documento de especificación del dominio y requisitos del sistema de préstamos.

<details>
<summary>Pistas de conocimiento</summary>

- Considerar los diferentes tipos de préstamos y sus requisitos específicos.
- Analizar posibles edge cases y cómo manejarlos.

</details>

### Fase 2: Evaluación de decisiones de arquitectura

**Objetivo:** Evaluar y seleccionar decisiones de arquitectura que cumplan con los atributos de calidad medibles y explicitar los trade-offs.

**Tiempo estimado:** 10 horas

**Instrucciones:**

- Identificar al menos dos decisiones de arquitectura controversiales para el sistema de préstamos.
- Evaluar cada decisión en términos de sus ventajas, desventajas y trade-offs.
- Seleccionar la mejor opción para cada decisión y justificar la elección con atributos de calidad medibles (ej. throughput, latencia, disponibilidad).

**Entregable:** Documento de evaluación de decisiones de arquitectura con trade-offs explicitos.

<details>
<summary>Pistas de conocimiento</summary>

- Considerar la consistencia, disponibilidad y partición (CAP) en las decisiones de arquitectura.
- Analizar el impacto de las decisiones en la escalabilidad y mantenibilidad del sistema.

</details>

### Fase 3: Comunicación de decisiones a audiencias distintas

**Objetivo:** Comunicar las decisiones de arquitectura a audiencias técnicas y de negocio de manera efectiva.

**Tiempo estimado:** 10 horas

**Instrucciones:**

- Preparar una presentación para comunicar las decisiones de arquitectura a una audiencia técnica.
- Preparar un informe para comunicar las decisiones de arquitectura a una audiencia de negocio.
- Asegurarte de que ambas audiencias puedan tomar decisiones informadas sin pedir aclaraciones adicionales.

**Entregable:** Presentación y informe de decisiones de arquitectura para audiencias técnicas y de negocio.

<details>
<summary>Pistas de conocimiento</summary>

- Utilizar lenguaje y ejemplos relevantes para cada audiencia.
- Incluirse en la presentación y el informe los atributos de calidad medibles y los trade-offs explicitos.

</details>

### Fase 4: Implementación y verificación de la solución

**Objetivo:** Implementar y verificar que la solución cumple con los requisitos y decisiones de arquitectura seleccionadas.

**Tiempo estimado:** 10 horas

**Instrucciones:**

- Implementar la solución siguiendo las decisiones de arquitectura seleccionadas.
- Verificar que la solución cumple con los requisitos y decisiones de arquitectura mediante pruebas y métricas.
- Documentar cualquier desviación o ajuste realizado durante la implementación.

**Entregable:** Solución implementada y verificada con documentación de desviaciones o ajustes.

<details>
<summary>Pistas de conocimiento</summary>

- Utilizar pruebas unitarias, de integración y de sistema para verificar la solución.
- Monitorear métricas de rendimiento y disponibilidad durante la implementación.

</details>

## Dimensiones Evaluadas

- **queEs**: ¿Qué es una decisión de arquitectura y por qué es importante en este reto?
- **paraQueSirve**: ¿Para qué sirve evaluar y seleccionar decisiones de arquitectura en este contexto?
- **comoSeUsa**: ¿Cómo se usa un trade-off para justificar una decisión de arquitectura?
- **erroresComunes**: ¿Cuáles son los errores comunes al comunicar decisiones de arquitectura a diferentes audiencias?
- **queDecisionesImplica**: ¿Qué decisiones implica la implementación de una solución de préstamos con los requisitos y decisiones de arquitectura seleccionados?

## Criterios de Evaluacion

- Definir claramente los actores, fuentes, sumideros y reglas de negocio para el sistema de préstamos.
- Evaluar y seleccionar decisiones de arquitectura que cumplan con los atributos de calidad medibles y explicitar los trade-offs.
- Comunicar efectivamente las decisiones de arquitectura a audiencias técnicas y de negocio.
- Implementar y verificar que la solución cumple con los requisitos y decisiones de arquitectura seleccionadas.

## Como trabajar con un asistente de IA

Hay dos caminos, elegi uno:

- **AGENTS.md** (recomendado) — instrucciones nativas del repo. Abri esta carpeta con tu agente local (Claude Code, Cursor, Codex, Copilot, Gemini) y las carga solo. Sabe que archivos faltan y con que comando se verifica, y completa el scaffold escribiendo en disco.
- **PROMPT_MEJORA.md** — para copiar y pegar en un chat (claude.ai, ChatGPT). Devuelve un ZIP con el proyecto. Sirve si no tenes un agente en el IDE.

Ninguno de los dos resuelve las fases del reto: eso es tu trabajo.

## Verificacion

El proyecto esta listo para trabajar cuando este comando corre sin errores:

```bash
python3 -c "import glob,sys; assert glob.glob('adr/*.md') and glob.glob('diagramas/*.mmd'), 'faltan ADRs o diagramas'"
```

---

*Reto generado automaticamente por Challenge Generator - Pragma*
