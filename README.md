# restart_huggingface_DR

Mantiene vivo el Hugging Face Space [`ferferefer/DR`](https://huggingface.co/spaces/ferferefer/DR)
(Diabetic Retinopathy Detection App) reiniciandolo automaticamente con GitHub Actions
cuando se duerme o se cae.

## Como funciona

El workflow [`.github/workflows/restart-space.yml`](.github/workflows/restart-space.yml) se ejecuta:

- **Cada 30 minutos** mediante un cron (`*/30 * * * *`, hora UTC).
- **Manualmente** desde la pestana *Actions* -> *Restart Hugging Face Space* -> *Run workflow*.

En cada ejecucion:

1. Consulta el estado del Space con `HfApi.get_space_runtime`.
2. Si el estado es `RUNNING`, `BUILDING`, `APP_STARTING` (o similares), no hace nada.
3. Si esta dormido o caido (`SLEEPING`, `PAUSED`, `RUNTIME_ERROR`, `STOPPED`...), llama a
   `HfApi.restart_space` para levantarlo.

Todo corre en `ubuntu-latest`, dentro de los minutos gratuitos de GitHub Actions
(el repositorio es publico, asi que el uso de Actions no se factura).

## Configuracion requerida

El token **nunca** se escribe en el codigo: se lee de un secreto del repositorio.

1. Genera un token de escritura en https://huggingface.co/settings/tokens
   (permisos: *Write*, o *Fine-grained* con acceso de escritura al Space `ferferefer/DR`).
2. En este repo ve a **Settings -> Secrets and variables -> Actions -> New repository secret**.
3. Nombre: `HF_TOKEN`
4. Valor: el token generado.

Sin ese secreto el workflow falla en el primer paso con un mensaje claro.

## Notas

- El cron de GitHub Actions es "best effort": puede retrasarse unos minutos en horas de mucha carga.
- GitHub deshabilita los workflows programados en repos publicos tras **60 dias sin actividad**.
  Si eso pasa, basta con entrar a *Actions* y pulsar *Enable workflow*, o hacer cualquier commit.
- Para apuntar a otro Space, cambia la variable `SPACE_ID` en el workflow.
