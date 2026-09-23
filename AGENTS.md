# AGENTS.md

Instrucciones para el agente de IA que abra este repositorio (Claude Code, Cursor, Codex, Copilot, Gemini). Se cargan solas: no hay que pegar nada en ningun chat.

## Que es este repositorio

Es el codigo base de un reto de aprendizaje de Pragma: **Diseño de solución extremo a extremo para sistema de préstamos**.

| | |
|---|---|
| Tema | Diseno de solucion extremo a extremo 1790190135969 |
| Nivel | senior-l2 |
| Chapter | Arquitectura |
| Especialidad | Soluciones |
| Stack | Markdown / ADR (Architecture Decision Records) + C4 Model |
| Patron arquitectonico | Documentación arquitectónica con ADRs, C4 Model y atributos de calidad medibles |
| Tiempo estimado | 40 horas |

## Receta del stack

Esqueleto obligatorio:

- `adr/`
- `arquitectura.md`
- `diagramas/*.mmd`
- `atributos-de-calidad.md`

Dependencias:

- Mermaid CLI n/a
- C4 Model n/a
- OpenAPI 3.1 3.1.0
- AsyncAPI 2.6 2.6.0
- ADR Template n/a

## Tu tarea

Dejar este conjunto de artefactos en estado **verificable**: que el comando de verificacion corra sin errores. Escribi los archivos en disco, en este repositorio. No generes ZIPs ni archivos adjuntos.

En orden:

1. Corre `python3 -c "import glob,sys; assert glob.glob('adr/*.md') and glob.glob('diagramas/*.mmd'), 'faltan ADRs o diagramas'"` y mira que falla.
2. Completa lo que falte de la lista de abajo: manifiesto de dependencias, punto de entrada, capa de interfaz y las capas del patron declarado.
3. Arregla SOLO los errores que impiden compilar o arrancar.
4. Volve a correr `python3 -c "import glob,sys; assert glob.glob('adr/*.md') and glob.glob('diagramas/*.mmd'), 'faltan ADRs o diagramas'"` hasta que pase.
5. Pará ahí.

## Regla dura: las fases son trabajo del humano

**PROHIBIDO implementar los entregables de las fases.** El valor del reto esta en que la persona los resuelva. Tu trabajo es que tenga un proyecto que arranca; el hueco pedagogico se queda como esta.

No resuelvas nada de esto:

- **Fase 1 — Especificación del dominio y requisitos**: Documento de especificación del dominio y requisitos del sistema de préstamos.
- **Fase 2 — Evaluación de decisiones de arquitectura**: Documento de evaluación de decisiones de arquitectura con trade-offs explicitos.
- **Fase 3 — Comunicación de decisiones a audiencias distintas**: Presentación y informe de decisiones de arquitectura para audiencias técnicas y de negocio.
- **Fase 4 — Implementación y verificación de la solución**: Solución implementada y verificada con documentación de desviaciones o ajustes.

Distincion operativa:

- **Arreglar** (si): import faltante, tipo que no existe, dependencia sin declarar, error de sintaxis, archivo referenciado que no existe.
- **No tocar** (no): logica de negocio incompleta, validaciones ausentes, secretos hardcodeados, APIs deprecadas que funcionan, concurrencia insegura, patrones mejorables. Eso es lo que la persona tiene que encontrar.

## Superficie de practica (NO completes)

Estos archivos SON el ejercicio de la persona. No los implementes; deja stubs. No toques la logica que el reto pide completar.

- [ ] `adr/001-idempotencia-solicitudes.md` — El topic pide decisiones de arquitectura: el ADR es el ejercicio.
- [ ] `adr/002-manejo-latencia-buro.md` — El topic pide decisiones de arquitectura: el ADR es el ejercicio.
- [ ] `adr/003-throughput-sla.md` — El topic pide decisiones de arquitectura: el ADR es el ejercicio.

## Lo que falta y tenes que completar

No se detectaron huecos: estan los archivos declarados, el boilerplate del stack y ninguna referencia quedo colgando. Igual corre el comando de verificacion — que los archivos existan no garantiza que compilen.

### Presentes (14)

- `adr/001-idempotencia-solicitudes.md`
- `adr/002-manejo-latencia-buro.md`
- `adr/003-throughput-sla.md`
- `arquitectura.md`
- `atributos-de-calidad.md`
- `riesgos-y-mitigaciones.md`
- `presentacion-tecnica.md`
- `informe-negocio.md`
- `verificacion-implementacion.md`
- `diagramas/contexto.mmd`
- `diagramas/contenedores.mmd`
- `diagramas/secuencia-solicitud.mmd`
- `contratos/openapi-prestamos.yaml`
- `contratos/asyncapi-auditoria.yaml`

### Capas del patron declarado

Cada una tiene que existir como directorio real con al menos un archivo. Codigo plano en la raiz no satisface el patron.

- `adr`
- `diagramas`
- `contratos`
- `documentos`

## Verificacion

```bash
python3 -c "import glob,sys; assert glob.glob('adr/*.md') and glob.glob('diagramas/*.mmd'), 'faltan ADRs o diagramas'"
```

El comando tiene que pasar SIN implementar los archivos de la superficie de practica: solo andamiaje.

Ese comando pasando es la definicion de "terminado" para vos.

## Convenciones que tenes que respetar

- Un solo ecosistema: no declares librerias de otro lenguaje ni mezcles gestores de paquetes.
- Toda libreria que uses tiene que estar declarada en el manifiesto de dependencias.
- Todo import declarado tiene que usarse; todo tipo usado tiene que existir o venir de una dependencia declarada.
- El patron es **Documentación arquitectónica con ADRs, C4 Model y atributos de calidad medibles**: los contratos (interfaces, puertos) los define la capa interna y los implementa la externa, nunca al revés.
- Los archivos que crees llevan implementacion real, no stubs: sin `TODO`, sin cuerpos vacios, sin `// getters y setters`.

## Contexto del candidato

Sirve para calibrar el nivel del codigo, no para resolver las fases.

- Perfil: Chapter Arquitectura, Especialidad Soluciones, Senior
- Brecha que el reto ataca: Sostiene las decisiones de arquitectura con atributos de calidad medibles y trade-offs explicitos
- Mision: Candidato con 8 años de experiencia

---

*Generado por Challenge Generator — Pragma. `README.md` tiene el enunciado completo del reto para la persona. `PROMPT_MEJORA.md` es la variante para pegar en un chat, si se prefiere ese flujo.*
