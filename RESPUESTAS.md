Nombre: Mariana Lizzet Maay Pech

correo: Le21080470@merida.tecnm.mx	





¿Por qué los valores de rendimiento obtenidos entre dos VMs en el mismo host son mucho más altos que los que se obtendrían entre dos equipos físicos en una LAN? ¿Qué parte de la infraestructura de red queda fuera de esta prueba?

R= La comunicación entre máquinas virtuales alojadas en el mismo host no pasa por tarjetas de red físicas ni cables de red; el tráfico se transfiere internamente mediante copias de memoria RAM manejadas por el hipervisor (switch virtual en el kernel del SO). Quedan fuera las tarjetas de interfaz de red físicas (NICs), los cables Ethernet (UTP/Fibra), los puertos de switches físicos, routers, así como las atenuaciones, interferencias electromagnéticas y la latencia propia del medio físico de transmisión.

¿Cuál es la ventaja de usar Vagrant y una Vagrantfilepara esta práctica en lugar de crear las VM manualmente en VirtualBox? ¿Qué concepto de administración de TI representa esto?

R= Permite automatizar por completo el despliegue, aprovisionamiento e interconexión de las máquinas virtuales mediante código. Evita errores manuales en la interfaz gráfica, garantiza un entorno idéntico e intermitente y facilita compartir la configuración con un solo archivo. Representa el concepto de Infraestructura como Código (IaC) (Infrastructure as Code).

El Vagrantfileusa private\_networkcon IP estáticas. ¿Qué pasaría si usara type: "dhcp"en lugar de IP estática? ¿Cómo afectaría eso al comando iperf3 -c <IP>?

R= l iniciar las VMs, un servidor DHCP virtual asignaría direcciones IP de forma dinámica, las cuales podrían cambiar en cada reinicio o despliegue (vagrant up). El comando iperf3 -c <IP> fallaría o requeriría un paso previo para verificar manualmente qué dirección IP recibió el servidor antes de poder ejecutar la prueba, impidiendo la automatización del proceso de pruebas.

Comparando los resultados de la prueba UDP con los valores SLA de referencia (jitter < 30 ms, package loss < 1%), ¿tu red virtual los cumple? ¿Para qué tipo de aplicaciones reales serán suficientes esos valores?

R= Sí, los cumple sobradamente. En la prueba se obtuvo un jitter de 0.021" ms"  (muy por debajo de los 30" ms" ) y 0% de pérdida de paquetes (por debajo del 1%). Son excelentes para servicios en tiempo real altamente sensibles como telefonía IP (VoIP), videoconferencias HD (Zoom, Teams), streaming de audio/video en vivo y videojuegos multijugador en línea.



¿En qué escenarios de la vida real sería útil el enfoque de esta práctica (levantar entornos virtuales con Vagrant) para un administrador de redes? Da al menos dos ejemplos concretos.

R= Probar reglas de firewall, configuraciones de enrutamiento o servidores de red en un entorno aislado antes de desplegar las configuraciones en un entorno de producción real.



