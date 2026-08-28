<div align="center">
<img src="./assets/header_esp.png" alt="GitHub Copilot - Programa de Adopción" width="100%" />

# ⚙️ Workshop 3 — GitHub Copilot CLI: Automatización, Seguridad y Flujo End-to-End

**Hands-on de `copilot -p`, GitHub Actions, `/fleet`, análisis automático y feature end-to-end**

**⏱️ Duración total: 90 minutos**

</div>

---

Este workshop transforma el contenido de automatización, seguridad y cierre en una sesión de 90 minutos orientada a ejecución. El participante usa **Copilot CLI en modo headless**, lo integra a un flujo de GitHub Actions, prueba paralelismo con `/fleet`, construye un script de revisión automática y termina con una actividad end-to-end sobre una API Express.

> [!IMPORTANT]
> Los bloques teóricos de seguridad se convierten aquí en controles concretos aplicados durante los laboratorios. El objetivo es practicar comportamientos seguros, no repetir las slides conceptuales.

---

## 📋 Tabla de Contenidos

1. [🎯 Objetivos de aprendizaje](#-objetivos-de-aprendizaje)
2. [⏱️ Duración estimada](#️-duración-estimada)
3. [📚 Requisitos previos](#-requisitos-previos)
4. [🤖 Paso 1. Practicar `copilot -p` en modo headless](#-paso-1-practicar-copilot--p-en-modo-headless)
5. [🔁 Paso 2. Crear un Daily AI Summary con GitHub Actions](#-paso-2-crear-un-daily-ai-summary-con-github-actions)
6. [🧠 Paso 3. Paralelizar trabajo con `/fleet`](#-paso-3-paralelizar-trabajo-con-fleet)
7. [🔍 Paso 4. LAB 4 — Script de análisis automático](#-paso-4-lab-4--script-de-análisis-automático)
8. [🚀 Paso 5. Challenge end-to-end — agregar Swagger a la API](#-paso-5-challenge-end-to-end--agregar-swagger-a-la-api)
9. [🔐 Paso 6. Hardening y revisión de seguridad](#-paso-6-hardening-y-revisión-de-seguridad)
10. [✅ Checklist de finalización](#-checklist-de-finalización)
11. [🧭 Reflexiones finales](#-reflexiones-finales)

---

## 🎯 Objetivos de aprendizaje

Al finalizar este taller serás capaz de:

- ✅ Ejecutar GitHub Copilot CLI de forma no interactiva con `copilot -p`.
- ✅ Aplicar `--no-ask-user`, `--silent`, `--output-format`, `--allow-tool` y `--deny-tool` según el tipo de automatización.
- ✅ Integrar un prompt headless dentro de un workflow de GitHub Actions.
- ✅ Entender cuándo una tarea tiene paralelismo natural y utilizar `/fleet` para despachar subagentes.
- ✅ Monitorear `/tasks` y `/usage` durante una ejecución multiagente.
- ✅ Construir un script `scripts/ai-review.sh` para seguridad, complejidad ciclomática y tests.
- ✅ Ejecutar un flujo end-to-end basado en `/plan → revisión → Autopilot → validación`.
- ✅ Aplicar controles de seguridad antes de delegar tareas en repositorios sensibles.

---

## ⏱️ Duración estimada

| Bloque | Tiempo |
| --- | ---: |
| 🤖 `copilot -p` y flags esenciales | 10 min |
| 🔁 Daily AI Summary en GitHub Actions | 15 min |
| 🧠 `/fleet` y monitoreo de subagentes | 15 min |
| 🔍 LAB 4 — `ai-review.sh` | 20 min |
| 🚀 Challenge end-to-end con Swagger | 20 min |
| 🔐 Hardening y checklist de seguridad | 10 min |
| **Total** | **90 min** |

> [!NOTE]
> Los tiempos adaptan una sesión original con demos y discusión a un formato de laboratorio. La prioridad es que cada participante ejecute al menos una vez cada patrón de trabajo.

---

## 📚 Requisitos previos

Antes de comenzar:

- GitHub Copilot CLI instalado y autenticado.
- Repositorio del workshop clonado localmente.
- Acceso a Git y a los tests del proyecto.
- Acceso a GitHub Actions para el repositorio de práctica.
- Para el ejercicio headless en Actions: un **fine-grained PAT** con el permiso indicado en el material (`Copilot Requests`) y posibilidad de almacenarlo como secret del repositorio.
- `AGENTS.md` disponible si se completó el workshop anterior.

> [!WARNING]
> Utilizá únicamente credenciales y permisos autorizados para el workshop. No copies tokens dentro de archivos versionados.

---

## 🤖 Paso 1. Practicar `copilot -p` en modo headless

**⏱️ 10 minutos**

**Objetivo:** ejecutar una tarea one-shot sin abrir una sesión interactiva.

### 1.1 Primer comando headless

Desde la raíz del repositorio:

```bash
copilot -p "Revisa vulnerabilidades en src/"
```

Observá que el comando ejecuta el prompt y finaliza.

### 1.2 Evitar esperas interactivas

Probá:

```bash
copilot -p "Revisa vulnerabilidades en src/" --no-ask-user
```

> [!IMPORTANT]
> En el material, `--no-ask-user` es crítico para CI/CD: un proceso sin TTY no debe quedar esperando una respuesta humana.

### 1.3 Comparar formatos y permisos

Ejemplos del material:

```bash
copilot -p "tarea" --silent
```

```bash
copilot -p "tarea" --output-format=json
```

```bash
copilot -p "tarea" --allow-tool='shell(git:*)' --allow-tool='read'
```

```bash
copilot -p "tarea" --deny-tool='shell(rm)' --deny-tool='git(push)'
```

### Checkpoint

- [ ] Ejecutaste un prompt headless.
- [ ] Probaste `--no-ask-user`.
- [ ] Identificaste al menos una tool que conviene permitir y una que conviene denegar para tu caso.

---

## 🔁 Paso 2. Crear un Daily AI Summary con GitHub Actions

**⏱️ 15 minutos**

**Objetivo:** ejecutar Copilot CLI dentro de un runner para resumir cambios del repositorio.

### 2.1 Crear el workflow

Creá:

```text
.github/workflows/daily-summary.yml
```

Usá como base el workflow presentado en las slides:

```yaml
name: Daily AI Summary

on:
  schedule:
    - cron: '30 17 * * *'

jobs:
  summary:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5

      - run: npm install -g @github/copilot

      - env:
          COPILOT_GITHUB_TOKEN: ${{ secrets.PAT }}
        run: |
          copilot -p "Resume cambios de hoy, escribe en summary.md" \
            --allow-tool='shell(git:*)' \
            --allow-tool=write \
            --no-ask-user
```

### 2.2 Configurar el secret

Creá en el repositorio un secret llamado:

```text
PAT
```

Su valor debe ser el fine-grained PAT autorizado para el laboratorio.

### 2.3 Ejecutar durante el workshop

El workflow de la slide utiliza un `schedule`. La transcripción también menciona lanzarlo manualmente mediante un dispatcher.

Para completar el ejercicio dentro del timebox, el instructor puede habilitar una ejecución manual equivalente en el workflow o disparar la ejecución utilizando el mecanismo autorizado en el repositorio.

> [!NOTE]
> El material describe que `summary.md` puede utilizarse como salida/artefacto, pero la slide disponible no incluye el paso concreto de publicación del artefacto. Este workshop no inventa ese paso: validá primero que `summary.md` sea generado durante el job y aplicá el estándar de artifacts de tu organización si necesitás persistirlo.

### Resultado esperado

El job debe:

1. Hacer checkout del repositorio.
2. Instalar Copilot CLI.
3. Ejecutar el prompt en modo headless utilizando el secret.
4. Generar `summary.md` en el workspace del runner.

### Checkpoint

- [ ] Workflow creado.
- [ ] Secret `PAT` configurado.
- [ ] El job pudo ejecutar `copilot -p` o quedó preparado hasta el límite de permisos del entorno.
- [ ] Se identificó dónde validar la salida del agente.

---

## 🧠 Paso 3. Paralelizar trabajo con `/fleet`

**⏱️ 15 minutos**

**Objetivo:** separar una tarea en tracks independientes y observar cómo el agente principal orquesta subagentes.

### 3.1 Planificar antes de paralelizar

El material recomienda la secuencia:

```text
/plan → revisar el plan → verificar paralelismo natural → /fleet
```

Elegí una tarea que pueda dividirse sin que todos los tracks modifiquen el mismo archivo.

### 3.2 Ejecutar el ejemplo de feature

El material propone:

```text
/fleet Implementa el módulo de notificaciones:
- Track 1: servicio Node.js con SendGrid
- Track 2: tests unitarios con mocks
- Track 3: documentación de la API en docs/notifications.md
```

Si el repositorio ya contiene parte de esa funcionalidad, adaptá el ejercicio a otro feature del mismo tipo siempre que los tracks sigan siendo independientes.

### 3.3 Monitorear subagentes

Ejecutá:

```text
/tasks
```

Luego:

```text
/usage
```

Observá:

- Cuántos subagentes fueron despachados.
- Qué tarea realiza cada uno.
- Qué tareas tuvieron que esperar por dependencias.
- El consumo reportado por la sesión.

### Resultado esperado

El agente principal actúa como orquestador y separa trabajo que puede realizarse en paralelo.

> [!WARNING]
> El material destaca que no existe file locking entre subagentes. Si dos tracks modifican el mismo archivo, pueden aparecer conflictos al converger. Elegí tracks realmente independientes.

> [!TIP]
> Si `/fleet` no identifica bien el paralelismo, pedile explícitamente: `Descompone esto en tracks independientes, luego ejecútalos en paralelo`.

---

## 🔍 Paso 4. LAB 4 — Script de análisis automático

**⏱️ 20 minutos**

**Objetivo:** crear un script ejecutable que use Copilot en modo headless para automatizar tres revisiones del repositorio.

El material define tres tareas obligatorias:

```text
1. Análisis de seguridad.
2. Detección de funciones con alta complejidad ciclomática.
3. Verificación de que los tests pasan.
```

### 4.1 Crear el archivo

```bash
mkdir -p scripts
touch scripts/ai-review.sh
```

En Windows podés crear el archivo desde VS Code o desde la terminal equivalente disponible.

### 4.2 Pedir al agente que construya el script

En Copilot CLI, utilizá un prompt equivalente a:

```text
Crea scripts/ai-review.sh para ejecutar una revisión automática de este repositorio usando GitHub Copilot CLI en modo headless.

El script debe realizar, en este orden:
1. Un análisis de seguridad sin permitir escritura y sin pedir input interactivo.
2. Un análisis para detectar funciones con alta complejidad ciclomática.
3. Una verificación de que los tests del proyecto pasan, permitiendo la tool shell cuando sea necesaria.

Al finalizar debe dejar un summary legible con los resultados de las tres verificaciones.
No agregues tareas diferentes a estas tres.
```

### 4.3 Verificar los controles obligatorios

Revisá que la sección de seguridad utilice los flags requeridos por la slide:

```text
--deny-tool='write'
--no-ask-user
```

Y que la ejecución de tests utilice el control indicado:

```text
--allow-tool='shell'
```

> [!NOTE]
> El material define el objetivo y los flags, pero no entrega un contenido canónico completo de `ai-review.sh`. Por eso el participante debe generar y revisar el script con el agente en lugar de copiar una solución prefabricada.

### 4.4 Ejecutar

```bash
chmod +x scripts/ai-review.sh
./scripts/ai-review.sh
```

### 4.5 Revisar el summary

Confirmá que podés distinguir claramente los resultados de:

- Seguridad.
- Complejidad ciclomática.
- Tests.

### Bonus

Si el resultado identifica tareas independientes de remediación, probá utilizar `/fleet` para despacharlas en paralelo.

### Checkpoint

- [ ] `scripts/ai-review.sh` creado.
- [ ] Análisis de seguridad ejecutado sin escritura.
- [ ] Complejidad ciclomática analizada.
- [ ] Tests ejecutados.
- [ ] Summary revisado.

---

## 🚀 Paso 5. Challenge end-to-end — agregar Swagger a la API

**⏱️ 20 minutos**

**Objetivo:** ejecutar un flujo completo de planificación, implementación, validación y documentación sobre la API Express del workshop.

La transcripción utiliza como feature realista **agregar Swagger/OpenAPI** a la aplicación.

### 5.1 Pedir primero la planificación

Entrá en Modo Plan y solicitá una implementación equivalente a:

```text
Analiza esta API Express y crea un plan para agregar Swagger/OpenAPI.
El plan debe cubrir las dependencias necesarias, el módulo/configuración OpenAPI, la documentación de las rutas existentes y la validación de que la aplicación y los tests continúan funcionando.
También debe actualizar AGENTS.md para que los nuevos endpoints/controladores mantengan la documentación Swagger correspondiente.
No implementes nada hasta que revise el plan.
```

### 5.2 Revisar scope

Verificá que el plan contemple los elementos observados en la demo:

- Dependencias necesarias para Swagger.
- Configuración o módulo OpenAPI.
- Documentación de rutas/controladores.
- Wiring de los endpoints.
- Validación de build/tests.
- Actualización de `AGENTS.md`.

### 5.3 Aceptar el plan y ejecutar

Una vez revisado el alcance, permití la implementación y, si el entorno de práctica lo permite, utilizá Autopilot para completar la tarea dentro del timebox.

### 5.4 Validar

Comprobá:

- Build correcto.
- Tests pasando o problemas preexistentes claramente diferenciados.
- Aplicación ejecutándose localmente.
- Swagger/OpenAPI accesible en el puerto/configuración generada para tu instancia.

### 5.5 Validar AGENTS.md

Revisá que la regla agregada sea consistente con el objetivo del laboratorio y no contradiga las convenciones existentes del repositorio.

### 5.6 Preparar PR

Pedí al agente que prepare un PR con una descripción basada en el cambio realizado.

### Evidencia del laboratorio

El material original pide entregar un **screenshot del Swagger levantado localmente**. Conservá esa misma evidencia para este workshop.

### Checkpoint

- [ ] Plan revisado antes de implementar.
- [ ] Swagger/OpenAPI agregado.
- [ ] Build/tests revisados.
- [ ] Swagger visible localmente.
- [ ] `AGENTS.md` actualizado.
- [ ] PR preparado o creado.
- [ ] Screenshot capturado.

---

## 🔐 Paso 6. Hardening y revisión de seguridad

**⏱️ 10 minutos**

**Objetivo:** aplicar los riesgos del módulo M5 como checklist operativo sobre lo que acabás de construir.

### 6.1 Verificar directorio de ejecución

Antes de cualquier sesión:

```bash
pwd
```

Confirmá que Copilot se ejecuta dentro del repositorio y no desde `~`.

### 6.2 Revisar Autopilot

Preguntate:

- ¿Este repositorio es sensible o productivo?
- ¿Realmente necesito Autopilot?
- ¿Debería usar Normal o Plan y limitar tools?

El material recomienda evitar Autopilot sin restricciones en repositorios sensibles.

### 6.3 Definir tools que deben bloquearse

Como mínimo, revisá si corresponde denegar:

```text
shell(rm)
git(push)
```

Y cualquier herramienta que pueda ejecutar operaciones irreversibles sobre infraestructura o datos.

### 6.4 Revisar MCP

Si existen MCP servers conectados a bases de datos u otros servicios, revisá si su alcance puede limitarse a operaciones de lectura cuando el laboratorio no necesita escritura.

### 6.5 Revisar políticas de organización

Si aparece un mensaje indicando que Copilot CLI está deshabilitado para la organización, el material indica que la habilitación depende de las policies de GitHub Copilot administradas por la organización.

### Checklist de hardening

- [ ] Sesión lanzada desde el directorio correcto.
- [ ] Autopilot utilizado únicamente donde corresponde.
- [ ] Tools destructivas bloqueadas cuando no son necesarias.
- [ ] MCP revisados con principio de mínimo privilegio.
- [ ] Tokens almacenados como secrets y no en el repositorio.
- [ ] Políticas organizacionales respetadas.

---

## ✅ Checklist de finalización

- [ ] Ejecutaste `copilot -p` con `--no-ask-user`.
- [ ] Creaste o revisaste un workflow de GitHub Actions con Copilot headless.
- [ ] Probaste `/fleet` con tracks independientes.
- [ ] Usaste `/tasks` y `/usage` para observar la ejecución paralela.
- [ ] Generaste y ejecutaste `scripts/ai-review.sh`.
- [ ] Revisaste seguridad, complejidad y tests desde el script.
- [ ] Completaste el challenge de Swagger/OpenAPI o llegaste a un checkpoint verificable dentro del timebox.
- [ ] Actualizaste o validaste `AGENTS.md`.
- [ ] Preparaste un Pull Request del cambio end-to-end.
- [ ] Aplicaste el checklist de seguridad final.

---

## 🧭 Reflexiones finales

- `copilot -p` permite mover capacidades agénticas desde la terminal interactiva hacia **scripts y pipelines no interactivos**.
- `--no-ask-user` evita que una automatización sin TTY quede esperando input humano.
- `/fleet` aporta velocidad solamente cuando la especificación tiene **tracks realmente independientes**.
- Un script de revisión con Copilot puede combinar análisis de seguridad, complejidad y ejecución de tests como una única tarea automatizable.
- El flujo **/plan → revisar scope → ejecutar → validar → PR** mantiene una etapa de control humano antes de delegar implementación.
- Seguridad y gobernanza deben aplicarse en cada ejecución: directorio correcto, permisos mínimos, tools restringidas y credenciales fuera del código.

> [!IMPORTANT]
> Automatizar no significa eliminar revisión. Cuanto más autónomo sea el flujo, más importante es definir correctamente alcance, tools, restricciones y criterios de validación.

---