Investigación: Implementación de Laboratorios Avanzados con GNS3, VirtualBox y ESXi
¡Hola! En este repositorio presento mi investigación sobre cómo montar un entorno de red profesional usando GNS3 sobre Windows 11. El objetivo es entender cómo conviven los hipervisores de Tipo 1 y Tipo 2 y cómo optimizar el rendimiento para que nuestras simulaciones no "exploten" la CPU.
1. Arquitectura de Virtualización en Windows 11
Aislamiento de Núcleo y VBS
Windows 11 viene con la Seguridad basada en Virtualización (VBS) activa por defecto. Como estudiantes, esto nos impacta directamente:
El "Búnker" de Seguridad: VBS crea un entorno aislado para proteger el kernel. Para lograrlo, Windows activa un hipervisor interno (Hyper-V) de forma silenciosa.
Impacto: Al estar Hyper-V "adueñándose" del hardware, otros programas como VirtualBox o VMware deben correr en un modo de "virtualización anidada", lo que puede causar que la GNS3 VM sea más lenta o que el soporte de KVM aparezca como False.
Activación de VT-x/AMD-V
Para que GNS3 funcione, la virtualización debe estar habilitada desde la "raíz" (el BIOS/UEFI).
En el BIOS: Buscamos términos como Intel Virtualization Technology o SVM Mode (en AMD) y los ponemos en Enabled.
Verificación: Desde Windows, la forma más fácil es abrir el Administrador de Tareas (Ctrl+Shift+Esc), ir a Rendimiento > CPU y verificar que diga Virtualización: Habilitado.
2. GNS3 VM: El Motor de Simulación
La GNS3 VM es donde ocurre la magia. Sin ella, estaríamos limitados a usar solo recursos básicos de Windows.
KVM (Kernel-based Virtual Machine)
KVM es un módulo de virtualización para el kernel de Linux (que corre dentro de la GNS3 VM).
¿Por qué es obligatorio que sea "True"? Si KVM está en True, las imágenes de routers y firewalls corren con aceleración de hardware nativa. Si está en False, se usa emulación pura, lo que hace que un simple router consuma el 100% de tu CPU real.
Configuración de Recursos
Para no desestabilizar mi laptop, sigo esta regla:
RAM: Asignar el 50% de la RAM disponible. Si tienes 16GB, dale 8GB a la VM.
vCPU: Asignar 2 o 4 núcleos. Nunca asignes todos los núcleos lógicos, porque Windows 11 se quedará "congelado" mientras haces tus configuraciones de red.
3. Integración con VirtualBox (Local)
Configuración de Red (Host-Only)
Para que el programa GNS3 (la interfaz que vemos) hable con el servidor (la VM), necesitamos un puente:
Creamos un Host-Only Network en VirtualBox (usualmente vboxnet0).
Esto crea un túnel privado entre mi Windows y la VM sin depender del Wi-Fi de mi casa.
Modo Promiscuo
Este concepto es clave. Por defecto, una tarjeta de red solo acepta paquetes dirigidos a su propia dirección MAC.
Técnicamente: Como en GNS3 vamos a pasar tráfico de Capa 2 (VLANs, STP, etc.), necesitamos que la tarjeta de red de la VM "escuche" todo el tráfico, no solo el suyo. Por eso configuramos el Modo Promiscuo en "Permitir todo".
4. Integración con VMware ESXi (Remoto)
Arquitectura Cliente-Servidor
Aquí es donde el laboratorio se vuelve "Pro". Mi laptop actúa solo como Cliente (GUI), enviando comandos a un servidor físico externo que corre ESXi.
Esto permite simular redes gigantescas (20+ routers) porque el procesamiento no ocurre en mi laptop, sino en el servidor remoto.
Seguridad en vSwitch
En ESXi, por seguridad, el switch virtual (vSwitch) bloquea cambios de MAC y tráfico desconocido. Para que GNS3 funcione, debemos entrar al Port Group y cambiar estas tres políticas:
Promiscuous Mode: Accept
MAC Address Changes: Accept
Forged Transmits: Accept
5.  Matriz de Solución de Errores (Troubleshooting)
A continuacion se les brindara los enlaces de lo utilizado y de las fuentes usadas.
Diagrama: https://lucid.app/lucidchart/816635e3-036a-4c79-9210-5b70a0ed944f/edit?viewport_loc=1834%2C-272%2C3262%2C1630%2C0_0&invitationId=inv_876bbe42-07d3-47b6-9bed-3e2e8eaf411d
Manual: https://docs.gns3.com/

