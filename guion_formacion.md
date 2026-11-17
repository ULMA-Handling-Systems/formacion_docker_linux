# Formación Linux y Docker: ejercicios prácticos

Dos bloques de ejercicios para 4 alumnos sin experiencia en Linux, pensados para un codespace con Ubuntu 24.04 y Docker en menos de 2 horas.

## Plan de la sesión

La sesión cabe en 110 minutos, con 10 de margen para imprevistos; el punto crítico es no pasar de 45 minutos en Linux.

| Bloque | Duración | Ejercicios |
| --- | --- | --- |
| Linux: orientarse, usuarios, directorios, ficheros | 25 min | 1 a 4 |
| Linux: vi y primer script | 20 min | 5 y 6 |
| Descanso | 5 min |  |
| Docker: comprobar entorno y `docker load` | 15 min | 7 y 8 |
| Docker: arrancar y supervisar contenedores | 30 min | 9 a 11 |
| Docker: limpieza y cierre | 10 min | 12 |

## Parte 1: fundamentos de Linux

Todos los ejercicios se hacen en la terminal del codespace (Ubuntu 24.04). Cada uno termina con una comprobación que el alumno puede hacer solo.

### Ejercicio 1: ¿dónde estoy y quién soy?

```bash
whoami                 # usuario actual
hostname               # nombre de la máquina
pwd                    # directorio actual
cat /etc/os-release    # versión de Ubuntu
uname -r               # versión del kernel
echo $SHELL            # intérprete de comandos
```

Comprobación: el alumno sabe decir su usuario, su directorio actual y que la distribución es Ubuntu 24.04.

### Ejercicio 2: usuarios y login

En Codespaces el login ya está hecho, así que se simula entrando como otro usuario.

```bash
sudo adduser formacion          # pide una contraseña; el resto de campos, Intro
su - formacion                  # entrar como formacion ("login" dentro de la sesión)
whoami                          # formacion
id                              # uid, gid y grupos
sudo ls /root                   # falla: formacion no está en sudoers
exit                            # volver al usuario original
sudo ls /root                   # funciona
```

Comprobación: entienden la diferencia entre un usuario normal y uno con `sudo`, y que `exit` cierra la sesión del usuario actual, no el terminal.

### Ejercicio 3: moverse por los directorios

```bash
cd ~                                  # ir al home
mkdir -p practica/docs practica/scripts practica/tmp
cd practica
ls -la                                # listado largo, incluye ocultos
cd docs
pwd                                   # /home/<usuario>/practica/docs
cd ..                                 # subir un nivel
cd scripts && pwd
cd -                                  # volver al directorio anterior
cd /etc && ls | head                  # ruta absoluta
cd ~/practica/tmp                     # ruta absoluta con ~
cd ../docs                            # ruta relativa
```

Comprobación: distinguen ruta absoluta (empieza por `/`) de relativa, y saben volver siempre al home con `cd`.

### Ejercicio 4: crear, copiar, mover y borrar ficheros

```bash
cd ~/practica/docs
touch a.txt b.txt
echo "hola linux" > a.txt            # escribe (sobrescribe)
echo "segunda línea" >> a.txt        # añade al final
cat a.txt
cp a.txt copia.txt
mv copia.txt ../tmp/                  # mover
mv b.txt c.txt                        # renombrar
ls . ../tmp
rm ../tmp/copia.txt
rm -r ~/practica/tmp                  # borrar directorio completo
ls ~/practica
```

Comprobación: `ls ~/practica` ya no muestra `tmp`. Conviene advertir de que `rm` no tiene papelera.

### Ejercicio 5: vi para crear, editar y guardar

vi tiene dos modos: **normal** (las teclas son comandos) e **inserción** (se escribe texto). Se pasa a inserción con `i` y se vuelve a normal con `Esc`. Todo el ejercicio gira en torno a eso.

| Tecla | Acción |
| --- | --- |
| `i` | entrar en modo inserción |
| `Esc` | volver a modo normal |
| `:w` | guardar |
| `:wq` | guardar y salir |
| `:q!` | salir sin guardar |
| `dd` | borrar la línea actual |
| `u` | deshacer |
| `/palabra` | buscar, `n` para el siguiente |
| `gg` y `G` | ir al principio y al final del fichero |

```bash
cd ~/practica/docs
vi notas.txt
```

1. Pulsa `i` y escribe tres líneas: `linea uno`, `linea dos`, `linea tres`.
2. Pulsa `Esc`, escribe `:wq` y Intro. Comprueba con `cat notas.txt`.
3. Vuelve a abrirlo con `vi notas.txt`. Mueve el cursor con las flechas a `linea dos`, pulsa `dd` para borrarla y `u` para recuperarla.
4. Con el cursor en `linea tres`, pulsa `i`, añade ` modificada` y guarda con `Esc` y `:w`.
5. Escribe una línea basura, pulsa `Esc` y sal con `:q!`. Comprueba con `cat` que el cambio no se guardó.

Comprobación: el fichero acaba con tres líneas y la tercera dice `linea tres modificada`. El error más habitual es escribir `:wq` sin haber pulsado `Esc`, y conviene que lo vean a propósito una vez.

### Ejercicio 6: primer script y permisos

```bash
cd ~/practica/scripts
vi hola.sh
```

Contenido del fichero:

```bash
#!/bin/bash
echo "Hola, soy $(whoami) en $(hostname)"
echo "Hoy es $(date +%d/%m/%Y)"
```

```bash
./hola.sh              # falla: Permission denied
ls -l hola.sh          # sin permiso de ejecución
chmod +x hola.sh
ls -l hola.sh          # ahora aparece la x
./hola.sh
```

Comprobación: el script se ejecuta y saben leer `-rwxr-xr-x`: tipo, permisos de propietario, grupo y resto.

## Parte 2: Docker, carga desde fichero y supervisión

Las imágenes no se descargan de Docker Hub: se cargan desde ficheros `.tar` que ya están en el repo, en la carpeta `imagenes/`. Es el mismo procedimiento que se usa en un entorno sin acceso a un registry.

### Ejercicio 7: comprobar el entorno

```bash
docker version         # cliente y servidor (daemon) responden
docker info | head -20
docker images          # vacío: no hay ninguna imagen cargada
docker ps -a           # vacío: no hay contenedores
```

Comprobación: ven que el cliente (`docker`) y el daemon son dos piezas distintas, y que ambos están dentro de su codespace.

### Ejercicio 8: cargar imágenes desde fichero

```bash
ls -lh imagenes/
docker load -i imagenes/alpine-3.20.tar
docker load -i imagenes/nginx-alpine.tar
docker images
docker image inspect alpine:3.20 --format '{{.Id}} {{.Size}}'
```

Tras cada `load` debe aparecer `Loaded image: alpine:3.20` y `Loaded image: nginx:alpine`. Después se hace el camino inverso para entender que es una foto de la imagen, no un instantáneo del contenedor:

```bash
docker save -o /tmp/copia-alpine.tar alpine:3.20
ls -lh /tmp/copia-alpine.tar
```

Comprobación: `docker images` lista las dos imágenes y saben explicar la diferencia entre `docker load` / `docker save` (imágenes) y `docker import` / `docker export` (sistemas de ficheros de contenedores); esta última pareja se menciona sin practicarla.

### Ejercicio 9: arrancar contenedores

```bash
# Efimero: se ejecuta, imprime y se borra solo
docker run --rm alpine:3.20 echo "hola desde un contenedor"

# Interactivo: estar dentro de un contenedor
docker run --rm -it alpine:3.20 sh
  cat /etc/os-release   # Alpine, no Ubuntu
  exit

# En segundo plano, con nombre
docker run -d --name reloj alpine:3.20 sh -c 'while true; do date; sleep 2; done'
docker run -d --name web -p 8080:80 nginx:alpine
```

En la pestaña **Ports** del codespace aparece el puerto 8080; al abrirlo en el navegador sale la página de bienvenida de nginx.

Comprobación: distinguen `-it` (interactivo, en primer plano) de `-d` (segundo plano), y ven que el contenedor es Alpine mientras ellos están en Ubuntu.

### Ejercicio 10: supervisar contenedores

| Comando | Qué responde |
| --- | --- |
| `docker ps` / `docker ps -a` | qué está corriendo / qué existe, incluso parado |
| `docker logs --tail 5 -t reloj` | las últimas líneas de salida, con hora |
| `docker logs -f reloj` | salida en directo (`Ctrl+C` para salir) |
| `docker stats --no-stream` | CPU, memoria y red de cada contenedor |
| `docker top web` | procesos dentro del contenedor |
| `docker inspect web` | configuración y estado completos en JSON |
| `docker events` | eventos del daemon en tiempo real |

```bash
docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'
docker logs --tail 5 -t reloj
docker stats --no-stream
docker top web
docker inspect -f '{{.State.Status}} desde {{.State.StartedAt}}' web
docker exec -it web sh      # entrar en un contenedor que ya está corriendo
  ps
  exit
```

Para ver `docker events`, abren una segunda terminal (icono `+` del panel), lo dejan ejecutándose y en la primera hacen `docker stop reloj` y `docker start reloj`.

Comprobación: cada alumno sabe responder con un comando a "¿está corriendo?", "¿qué está escribiendo?" y "¿cuánta memoria consume?".

### Ejercicio 11: contenedores que se caen y reinicio automático

```bash
# Se cae con código de error 3 a los 5 segundos
docker run -d --name fallo alpine:3.20 sh -c 'echo arranco; sleep 5; exit 3'
sleep 6
docker ps -a --filter name=fallo
docker inspect -f 'estado={{.State.Status}} codigo={{.State.ExitCode}}' fallo
docker logs fallo

# Lo mismo, pero con política de reinicio
docker run -d --name resiste --restart on-failure:3 alpine:3.20 sh -c 'echo arranco; sleep 3; exit 1'
sleep 20
docker inspect -f 'reinicios={{.RestartCount}} estado={{.State.Status}}' resiste
docker logs resiste
```

En `fallo` aparece `Exited (3)`. En `resiste` el log repite `arranco` cuatro veces (la ejecución inicial y tres reintentos) y `RestartCount` vale 3. Una política `unless-stopped` o `always` en servicios que deben estar siempre arriba se menciona de palabra.

Ampliación opcional, para quien termine antes: healthcheck.

```bash
docker run -d --name web2 -p 8081:80 \
  --health-cmd 'wget -qO- http://localhost/ > /dev/null || exit 1' \
  --health-interval 5s nginx:alpine
sleep 10
docker ps --filter name=web2    # (healthy)
docker exec web2 nginx -s stop  # el proceso principal termina y el contenedor se para
```

### Ejercicio 12: limpieza

```bash
docker ps -a
docker stop reloj web web2 2>/dev/null
docker container prune -f     # borra todos los contenedores parados
docker system df              # espacio ocupado por imágenes y contenedores
docker rmi nginx:alpine       # borrar una imagen
docker images
```

Comprobación: `docker ps -a` está vacío y solo queda `alpine:3.20`. Cierran parando el codespace desde GitHub (*Code → Codespaces → ⋯ → Stop*).

## Notas para el formador

Estos ejercicios no se han ejecutado en un codespace real, así que haz una pasada completa con una cuenta personal el día antes. Dos cosas condicionan que todo funcione.

**1. `devcontainer.json` ajustado.** Respecto a la versión anterior se quitan los `docker pull` (el ejercicio 8 exige que no haya imágenes cargadas) y se instalan `vim` y `tree`, porque la imagen base puede traer solo un `vi` mínimo o ninguno; comprueba `which vi` en tu prueba.

```json
{
  "name": "Formacion Docker + Linux",
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu-24.04",
  "features": {
    "ghcr.io/devcontainers/features/docker-in-docker:2": {}
  },
  "hostRequirements": { "cpus": 2 },
  "postCreateCommand": "sudo apt-get update && sudo apt-get install -y vim tree"
}
```

**2. Los `.tar` de la carpeta `imagenes/` del repo.** Se generan una sola vez, desde tu propio codespace, y se suben al repo:

```bash
docker pull alpine:3.20
docker pull nginx:alpine
mkdir -p imagenes
docker save -o imagenes/alpine-3.20.tar alpine:3.20
docker save -o imagenes/nginx-alpine.tar nginx:alpine
ls -lh imagenes/
git add imagenes && git commit -m "Imágenes para la práctica" && git push
```

GitHub rechaza ficheros de más de 100 MB; si el de nginx se acerca, comprime con `docker save nginx:alpine | gzip > imagenes/nginx-alpine.tar.gz` y los alumnos lo cargan igual con `docker load -i`.

Problemas previsibles durante la sesión:

- **`docker info` falla justo al abrir el codespace.** El daemon tarda unos segundos en arrancar; basta con repetir el comando.
- **Alguien no ve el puerto 8080.** Se abre desde la pestaña *Ports*; si está vacía, el contenedor `web` no está corriendo (`docker ps`).
- **Alumno que se queda atascado en vi.** `Esc` y después `:q!` lo saca de cualquier situación; ponlo en la pizarra desde el principio.
- **Se quedan sin cuota de Codespaces.** Cada alumno debe usar su cuenta personal; en cuentas de organización no hay cuota gratuita.
