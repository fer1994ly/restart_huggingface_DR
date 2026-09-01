# restart_huggingface_DR

Mantiene vivo el Hugging Face Space [`ferferefer/DR`](https://huggingface.co/spaces/ferferefer/DR)
(Diabetic Retinopathy Detection App) usando GitHub Actions.

## Por que hace falta

El Space corre en hardware `cpu-basic` (gratis), que **se duerme tras 48 horas sin visitas**
(`gcTimeout: 172800` segundos). Segun la documentacion de Hugging Face, cualquier visita al
Space reinicia ese contador y, si ya estaba dormido, lo vuelve a arrancar.

## Como funciona

El workflow [`.github/workflows/restart-space.yml`](.github/workflows/restart-space.yml) se ejecuta:

- **Cada 30 minutos** mediante un cron (`*/30 * * * *`, hora UTC).
- **Manualmente** desde la pestana *Actions* -> *Keep Hugging Face Space Alive* -> *Run workflow*.

En cada ejecucion consulta el estado del Space y actua segun el caso:

| Estado | Accion |
|---|---|
| `RUNNING`, `BUILDING`, `APP_STARTING`... | **Ping HTTP** al Space. Reinicia el contador de 48h, asi que nunca llega a dormirse. |
| `SLEEPING`, `STOPPED` | **Ping HTTP**. Una visita lo despierta: mas rapido y barato que un restart. |
| `PAUSED`, `RUNTIME_ERROR`, `BUILD_ERROR` | **`restart_space` via API.** Una visita no arregla estos estados. |

La idea principal es el ping preventivo: el Space no se duerme nunca porque recibe una visita
cada 30 minutos. El `restart_space` queda solo como red de seguridad para cuando se rompe de verdad.

Todo corre en `ubuntu-latest`, dentro de los minutos gratuitos de GitHub Actions
(el repositorio es publico, asi que el uso de Actions no se factura).

## Configuracion requerida

El token **nunca** se escribe en el codigo: se lee de un secreto del repositorio.

1. Genera un token de escritura en https://huggingface.co/settings/tokens
   (permisos: *Write*, o *Fine-grained* con acceso de escritura al Space `ferferefer/DR`).
2. En este repo ve a **Settings -> Secrets and variables -> Actions -> New repository secret**.
3. Nombre: `HF_TOKEN`
4. Valor: el token generado.

El ping por si solo no necesita token (el Space es publico), pero si lo usa la consulta de
estado y el `restart_space` de emergencia.

## Notas

- El cron de GitHub Actions es "best effort": puede retrasarse unos minutos en horas de mucha carga.
  Con un margen de 48 horas, ese retraso es irrelevante.
- GitHub deshabilita los workflows programados en repos publicos tras **60 dias sin actividad**.
  Si eso pasa, basta con entrar a *Actions* y pulsar *Enable workflow*, o hacer cualquier commit.
- Para apuntar a otro Space, cambia `SPACE_ID` y `SPACE_URL` en el workflow.
