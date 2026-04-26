# Informe Ejecutivo
### Caso No. 2026-001 — Bomba Lógica en Entorno de Active Directory

**Instituto Tecnológico de las Américas (ITLA)**  
Asignatura: Informática Forense · Profesor: Cándido Noel Ramírez  
Integrantes: Branyel Pérez · Francis Vidal · Elian Leonardo  
Fecha de entrega: 24/04/2026

---

## Resumen de los Acontecimientos

La mañana del miércoles 23 de abril de 2026, los empleados de la empresa **Path Secure SRL** no pudieron iniciar sesión en sus equipos al comenzar la jornada laboral. Todas las contraseñas del dominio habían sido cambiadas de forma simultánea durante la madrugada. Ningún miembro del área de Seguridad Informática o Tecnología reportó haber realizado dicho cambio, por lo que se determinó que el hecho fue producto de un **ciberataque**.

El incidente se trata de un ataque dirigido mediante un troyano de acceso remoto (**NJRat v0.7d**) a la encargada del Departamento de Recursos Humanos, **Ana Pérez**. El día anterior, Ana recibió por correo electrónico un archivo adjunto que simulaba ser la nómina del departamento — documento que acostumbraba recibir en esas fechas. Al abrirlo, nunca tuvo acceso a la supuesta nómina, aunque no dio mayor importancia al hecho. Ese archivo, disfrazado de PDF, era en realidad el troyano que proporcionaría al atacante acceso remoto completo a su equipo.

Con el acceso remoto garantizado, y aunque sin privilegios elevados en el sistema, el atacante exploró los archivos del equipo de Ana en busca de información útil. Encontró un archivo con las **credenciales de administrador del Active Directory** almacenadas en texto plano. Usando esa información, escaló privilegios dentro del dominio.

Una vez dentro del dominio con privilegios de administrador, el atacante ejecutó su ataque: una **bomba lógica** implementada mediante una tarea programada oculta en el servidor, con un nombre diseñado para mimetizarse con procesos legítimos del sistema operativo. La tarea fue configurada para ejecutarse en la madrugada del miércoles y contenía un script encargado de **cambiar todas las contraseñas del dominio** de forma simultánea.

---

## Recomendaciones

- **Apagar el servidor** y realizar un análisis y saneamiento completo para eliminar cualquier amenaza remanente y evitar propagación.
- **Auditar los departamentos** de Ciberseguridad y Tecnología para determinar por qué el servidor presentaba un nivel de seguridad tan bajo ante este tipo de ataques, y elevar los controles de seguridad.
- **Investigar el origen** del archivo `passwords_temporal.txt` en el escritorio del servidor — determinar por qué la empleada Ana Pérez contaba con credenciales administrativas en su equipo.
- **Capacitar a todos los empleados** en identificación de correos phishing y técnicas de ingeniería social para evitar que vuelvan a ser víctimas de este tipo de ataques.

---

*Documento elaborado por el Grupo 6 — Informática Forense · ITLA · Santo Domingo, República Dominicana*
