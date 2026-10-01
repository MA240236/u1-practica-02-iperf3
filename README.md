# Práctica 1.4 — Medición de Ancho de Banda con iPerf3 usando Vagrant

**Asignatura:** Administración de Redes (SCA-1002)
**Grupo:** SCA1002_8SB | **Periodo:** AGO–DIC 2026
**Unidad:** 1 — Funciones de la Administración de Redes
**Subtema:** 1.4 Desempeño (FCAPS: *Performance Management*)
**Sesión:** Martes / Jueves 19:00 – 21:00 hrs
**Plataforma:** Windows 10/11 + Vagrant + VirtualBox (2 VMs Ubuntu 22.04)

## Objetivo

Levantar dos máquinas virtuales Linux con **Vagrant** sobre Windows, conectadas en una red privada, para medir el desempeño de red mediante **iPerf3**, identificando métricas de ancho de banda, jitter y pérdida de paquetes sin depender del sistema operativo del host.

## Competencia a desarrollar

> Configura y administra servicios de red para el uso eficiente y confiable de la infraestructura tecnológica de la organización.

Al finalizar la práctica el alumno será capaz de:

- Instalar y configurar Vagrant y VirtualBox en Windows.
- Definir un entorno multi-máquina con un `Vagrantfile`.
- Provisionar automáticamente iPerf3 en ambas VMs mediante shell scripts.
- Ejecutar pruebas TCP y UDP entre dos VMs en red privada.
- Interpretar métricas de throughput, jitter y packet loss.

## Materiales y requisitos

| Recurso | Descripción |
|---|---|
| PC con Windows 10/11 | 64-bit, mínimo 8 GB RAM, 20 GB libres en disco |
| VirtualBox 7.x | Proveedor de virtualización |
| Vagrant 2.4.9 | Gestor de entornos virtuales (HashiCorp) |
| Acceso a internet | Para descargar el box de Ubuntu |
| PowerShell o cmd | Como Administrador |

> [!WARNING]
> Verifica en BIOS/UEFI que la virtualización esté habilitada (`Intel VT-x` o `AMD-V`). Sin esto, VirtualBox no puede crear VMs de 64 bits.

> [!WARNING]
> Si tienes **Hyper-V** habilitado (WSL2, Docker Desktop), puede entrar en conflicto con VirtualBox. Se recomienda desactivarlo temporalmente o usar el proveedor Hyper-V de Vagrant en su lugar.

## Topología

```
                        Windows 10/11 (Host)

    VM: servidor                            VM: cliente
    Ubuntu 22.04                            Ubuntu 22.04

    iperf3 -s          <---- TCP/UDP ---->  iperf3 -c
    192.168.56.10                           192.168.56.11

           Red privada: 192.168.56.0/24 (VirtualBox host-only)
                       Puerto iPerf3: 5201
```

## Paso 1 — Instalar VirtualBox

1. Descarga VirtualBox desde la página oficial: `https://www.virtualbox.org/wiki/Downloads` (seleccionar **Windows hosts**).
2. Ejecuta el instalador `.exe` descargado con doble clic.
3. Acepta las opciones por defecto y completa la instalación.
4. Verifica la instalación abriendo `cmd` y ejecutando:

```cmd
"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" --version
```

Salida esperada: `7.x.x`

## Paso 2 — Instalar Vagrant

1. Descarga el instalador `.msi` oficial desde HashiCorp: `https://releases.hashicorp.com/vagrant/2.4.9/vagrant_2.4.9_windows_amd64.msi`
2. Ejecuta el instalador con las opciones por defecto.
3. Reinicia el equipo al terminar — Vagrant agrega su ruta al PATH del sistema y requiere reinicio para que tome efecto.
4. Verifica en un `cmd` o PowerShell nuevo:

```cmd
vagrant --version
```

Salida esperada: `Vagrant 2.4.9`

## Paso 3 — Crear la carpeta del proyecto

```cmd
mkdir C:\vagrant-iperf
cd C:\vagrant-iperf
```

> [!NOTE]
> Todos los archivos de esta práctica viven dentro de `C:\vagrant-iperf\`. El archivo de referencia está en `./Vagrantfile`, en la raíz del repositorio clonado; cópialo a `C:\vagrant-iperf\Vagrantfile`. Esta última es la ruta que editarás en Windows y `C:\vagrant-iperf\` es la carpeta desde la que ejecutarás Vagrant. Guarda `RESPUESTAS.md` y las evidencias para entregar en el repositorio clonado.

## Paso 4 — Revisar el Vagrantfile

Dentro de `C:\vagrant-iperf\`, copia el `Vagrantfile` de esta práctica (puedes usar el Bloc de Notas o cualquier editor de texto). Define dos VMs (`servidor` y `cliente`) en la red privada `192.168.56.0/24`, cada una con `iperf3` instalado automáticamente vía *shell provisioner*.

> [!NOTE]
> ¿Por qué `192.168.56.x`? VirtualBox reserva por omisión el rango `192.168.56.0/21` para redes host-only. Usar IPs fuera de ese rango produce el error `The IP address configured for the host-only network is not within the allowed ranges`.

> [!NOTE]
> El `Vagrantfile` usa la box `bento/ubuntu-22.04` en vez de la oficial `ubuntu/jammy64`. Esta última solo publica el provider VirtualBox para arquitectura `amd64` (Intel/AMD); si tu equipo es un Mac con Apple Silicon (M1/M2/M3), `vagrant up` fallaría con `VBoxManage: error: Cannot run the machine because its platform architecture x86 is not supported on ARM`. `bento/ubuntu-22.04` publica también `arm64`, así que el mismo `Vagrantfile` funciona igual en Windows, Mac Intel y Mac Apple Silicon sin cambios.

## Paso 5 — Levantar las máquinas virtuales

```cmd
cd C:\vagrant-iperf
vagrant up
```

Vagrant realiza automáticamente:

1. Descarga del box `bento/ubuntu-22.04` (~600 MB, solo la primera vez).
2. Creación de las dos VMs en VirtualBox.
3. Configuración de las interfaces de red.
4. Instalación de iPerf3 en ambas VMs mediante el script de provisioning.

Verifica el estado de ambas VMs:

```cmd
vagrant status
```

Salida esperada:

```
Current machine states:

servidor                  running (virtualbox)
cliente                   running (virtualbox)
```

## Procedimiento de pruebas

### Paso 6 — Conectarse a las VMs

Abre dos ventanas de `cmd` o PowerShell en `C:\vagrant-iperf\`.

Ventana 1 (servidor):

```cmd
cd C:\vagrant-iperf
vagrant ssh servidor
```

Ventana 2 (cliente):

```cmd
cd C:\vagrant-iperf
vagrant ssh cliente
```

Verifica conectividad desde el cliente al servidor:

```bash
ping -c 4 192.168.56.10
```

Salida esperada: `4 packets transmitted, 4 received, 0% packet loss`

### Paso 7 — Prueba básica TCP

Ventana 1 (servidor) — iniciar iPerf3 en modo servidor:

```bash
iperf3 -s
```

Ventana 2 (cliente) — ejecutar prueba TCP básica:

```bash
iperf3 -c 192.168.56.10
```

> [!NOTE]
> El throughput alto es normal: ambas VMs comparten el mismo host físico y la red es virtual, no hay cable real de por medio.

Registra resultados:

| Métrica | Valor obtenido |
|---|---|
| Bitrate sender | _________ Gbits/sec |
| Bitrate receiver | _________ Gbits/sec |
| Duración | 10 segundos |

### Paso 8 — Prueba TCP con flujos paralelos

Ventana 2 (cliente):

```bash
iperf3 -c 192.168.56.10 -P 4
```

Observa la línea `[SUM]` al final de la salida.

| Flujos | Bitrate total [SUM] |
|---|---|
| 1 flujo (Paso 7) | _________ Gbits/sec |
| 4 flujos | _________ Gbits/sec |

### Paso 9 — Prueba UDP: jitter y packet loss

Ventana 2 (cliente):

```bash
iperf3 -c 192.168.56.10 -u -b 1G -t 30
```

| Parámetro | Significado |
|---|---|
| `-u` | Protocolo UDP |
| `-b 1G` | Tasa objetivo 1 Gbps (ajustada para red virtual) |
| `-t 30` | Duración: 30 segundos |

Registra:

| Métrica | Valor obtenido | Valor SLA referencia |
|---|---|---|
| Jitter | _________ ms | < 30 ms |
| Packet Loss | _________ % | < 1 % |
| Bitrate UDP real | _________ Gbits/sec | ≈ objetivo |

### Paso 10 — Prueba en sentido inverso (download)

Ventana 2 (cliente):

```bash
iperf3 -c 192.168.56.10 -R
```

| Dirección | Bitrate |
|---|---|
| Cliente → Servidor (upload) | _________ Gbits/sec |
| Servidor → Cliente (download) | _________ Gbits/sec |

### Paso 11 — Prueba continua con reporte cada 5 segundos

Ventana 2 (cliente):

```bash
iperf3 -c 192.168.56.10 -t 60 -i 5
```

### Paso 12 — Guardar resultados en archivo

Dentro de la VM cliente:

```bash
iperf3 -c 192.168.56.10 -t 30 > /tmp/resultado_tcp.txt
iperf3 -c 192.168.56.10 -u -b 1G -t 30 > /tmp/resultado_udp.txt
cat /tmp/resultado_tcp.txt
cat /tmp/resultado_udp.txt
```

Para copiar los archivos al equipo Windows (ejecutar en `cmd`, fuera de la VM):

```cmd
vagrant scp cliente:/tmp/resultado_tcp.txt C:\vagrant-iperf\resultado_tcp.txt
vagrant scp cliente:/tmp/resultado_udp.txt C:\vagrant-iperf\resultado_udp.txt
```

> [!NOTE]
> Si `vagrant scp` no está disponible, instala el plugin primero: `vagrant plugin install vagrant-scp`

## Tabla de resultados consolidada

| Prueba | Protocolo | Bitrate | Jitter | Packet Loss |
|---|---|---|---|---|
| TCP básica (10 s) | TCP | _____ Gbits/sec | N/A | N/A |
| TCP 4 flujos | TCP | _____ Gbits/sec | N/A | N/A |
| UDP 1 Gbps (30 s) | UDP | _____ Gbits/sec | _____ ms | _____ % |
| Inverso / download | TCP | _____ Gbits/sec | N/A | N/A |
| 60 s continuo | TCP | _____ Gbits/sec | N/A | N/A |

## Paso 13 — Apagar y destruir las VMs

Al terminar, sal de las VMs con `exit` en cada ventana SSH, luego desde `cmd`:

```cmd
cd C:\vagrant-iperf
vagrant halt
```

O bien, para eliminar completamente las VMs:

```cmd
vagrant destroy -f
```

| Comando | Efecto |
|---|---|
| `vagrant halt` | Apaga las VMs, se pueden volver a encender con `vagrant up` |
| `vagrant suspend` | Guarda el estado en RAM (snapshot rápido) |
| `vagrant destroy -f` | Elimina las VMs y todos sus discos |

## Solución de problemas comunes

| Error | Causa | Solución |
|---|---|---|
| `VT-x/AMD-V hardware acceleration is not available` | Virtualización deshabilitada en BIOS | Habilitar Intel VT-x o AMD-V en BIOS |
| `IP address not within allowed ranges` | IP fuera del rango de VirtualBox | Usar IPs en `192.168.56.x` como está en el Vagrantfile |
| `vagrant: command not found` | Vagrant no está en el PATH | Reiniciar Windows después de instalarlo |
| `Box 'bento/ubuntu-22.04' not found` | Sin acceso a internet | Verificar conexión y reintentar `vagrant up` |
| `VBOX_E_PLATFORM_ARCH_NOT_SUPPORTED` | Box de arquitectura equivocada para el host (por ejemplo, intentar correr una VM `amd64` en un Mac Apple Silicon) | Confirmar que el `Vagrantfile` usa `bento/ubuntu-22.04` (publica `amd64` y `arm64`), no la oficial `ubuntu/jammy64` (solo `amd64`) |
| `Connection timeout` en `vagrant ssh` | VM no terminó de iniciar | Esperar 30 segundos y reintentar |
| `iperf3: error - unable to connect` | Servidor no iniciado | Ejecutar `iperf3 -s` en la VM servidor antes de lanzar el cliente |

## Preguntas complementarias

- ¿Por qué los valores de throughput obtenidos entre dos VMs en el mismo host son mucho más altos que los que se obtendrían entre dos equipos físicos en una LAN? ¿Qué parte de la infraestructura de red queda fuera de esta prueba?
- ¿Cuál es la ventaja de usar Vagrant y un `Vagrantfile` para esta práctica en lugar de crear las VMs manualmente en VirtualBox? ¿Qué concepto de administración de TI representa esto?
- El `Vagrantfile` usa `private_network` con IPs estáticas. ¿Qué pasaría si usara `type: "dhcp"` en lugar de IP estática? ¿Cómo afectaría eso al comando `iperf3 -c <IP>`?
- Comparando los resultados de la prueba UDP con los valores SLA de referencia (jitter < 30 ms, packet loss < 1%), ¿tu red virtual los cumple? ¿Para qué tipo de aplicaciones reales serían suficientes esos valores?
- ¿En qué escenarios de la vida real sería útil el enfoque de esta práctica (levantar entornos virtuales con Vagrant) para un administrador de redes? Da al menos dos ejemplos concretos.

## Reporte a entregar

- Portada con nombre(s), grupo y fecha.
- Contenido del `Vagrantfile` utilizado.
- Captura de pantalla de `vagrant status` con ambas VMs en `running`.
- Captura de pantalla de cada prueba ejecutada en la VM cliente.
- Archivos `.txt` con resultados exportados (Paso 12).
- Tabla de resultados consolidada completada.
- Respuestas a las preguntas complementarias.
- Conclusiones (mínimo 10 líneas).

**Formato:** PDF. **Entrega:** plataforma del curso, siguiente sesión de clase.

## Referencias

- Vagrant — Instalación oficial: `https://developer.hashicorp.com/vagrant/install`
- Vagrant — Documentación de redes privadas: `https://developer.hashicorp.com/vagrant/docs/networking/private_network`
- Vagrant — Multi-machine: `https://developer.hashicorp.com/vagrant/docs/multi-machine`
- VirtualBox — Descarga: `https://www.virtualbox.org/wiki/Downloads`
- iPerf3 — Manual oficial: `https://software.es.net/iperf/invoking.html`
- iPerf3 Windows builds (ar51an): `https://github.com/ar51an/iperf3-win-builds`

---

*Consulta también la versión en PDF/LaTeX de esta práctica en este mismo directorio.*

## Entrega en GitHub Classroom

> [!IMPORTANT]
> Crea un archivo llamado exactamente `RESPUESTAS.md` en la raíz de tu repositorio de GitHub Classroom (junto a `README.md`, en tu equipo anfitrión). Escribe al inicio tu **nombre completo** y **correo electrónico**. Incluye las respuestas a todas las preguntas complementarias, conservando su orden y copiando cada pregunta antes de su respuesta.
>
> Agrega `RESPUESTAS.md` a tu commit y súbelo a GitHub. Conserva también las capturas, resultados y el reporte PDF cuando la práctica los solicite. El `.gitignore` excluye `.vagrant/` y archivos temporales; entrega el `Vagrantfile` y las configuraciones que correspondan, junto con tus evidencias.
