# Proyecto Intermodular DAW2

## Web del centro educativo e Intranet de gestión docente

Proyecto Intermodular del CFGS de Desarrollo de Aplicaciones Web (DAW), curso 2026/2027.

> **Fuente principal:** Memoria Técnica del Proyecto Intermodular DAW.  
> **Referencia de API:** documento de endpoints de ejemplo del proyecto equivalente del Centro Médico Santa Rosa de Lima.

---

## 1. Descripción general

El proyecto consiste en desarrollar **dos productos digitales independientes** para el centro educativo:

1. **Web pública e informativa del centro**, dirigida a familias, alumnado y comunidad educativa.
2. **Intranet privada de gestión docente**, destinada al profesorado y al equipo directivo.

La memoria establece que ambos sistemas **no deben compartir base de datos, sesión ni autenticación**. El único punto de conexión previsto es un enlace desde la web hacia la intranet, por ejemplo mediante un botón `Acceso intranet`.

La intranet es una aplicación transaccional y protegida, mientras que la web pública está orientada a contenidos, noticias, documentos, enseñanzas, programas y trámites.

---

## 2. Participantes

> **Pendiente de completar con los nombres reales de la pareja/equipo.**

| Participante | Rol / responsabilidad | Módulo |
|---|---|---|
| `Nombre Apellido` | Desarrollo / análisis | Pendiente |
| `Nombre Apellido` | Desarrollo / análisis | Pendiente |

### Reparto recomendado

Antes de programar, cada participante debería tener asignados:

- módulos funcionales concretos;
- tareas de backend;
- tareas de frontend;
- modelo de datos;
- endpoints;
- pruebas;
- documentación;
- integración.

El reparto debe permitir que ambos conozcan la arquitectura completa, aunque cada persona sea responsable de determinadas partes.

---

## 3. Arquitectura global

```text
                         PROYECTO INTERMODULAR DAW2
                                    |
                 +------------------+------------------+
                 |                                     |
          WEB PÚBLICA                              INTRANET
        informativa                              transaccional
                 |                                     |
        PHP sin framework                         FastAPI
                 |                                     |
          MySQL / MariaDB                         MongoDB
                 |                                     |
       Backoffice con sesión                      Redis
                 |                                     |
          API /api/v1                         API /api/v1
                 |                                     |
        Frontend web                              PWA
                                                     |
                                      React o Vue + TypeScript
