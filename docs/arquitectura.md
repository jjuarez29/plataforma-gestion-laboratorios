# Arquitectura Técnica Propuesta

## Modelo General
**Híbrido**: Local (On-Premise) + Nube

## Componentes Principales

- **GitLab** → Código fuente y CI/CD
- **Harbor** → Registro privado de imágenes de contenedores
- **Keycloak** → Gestión de identidad y acceso
- **Proxmox VE** → Virtualización de hardware
- **Kubernetes** → Orquestación de contenedores
- **PostgreSQL + MinIO** → Base de datos y almacenamiento

## Flujo Recomendado
Estudiantes → Descargan imagen → Ejecutan localmente con Docker/Podman → Entorno idéntico al laboratorio.

# Comentario sobre `arquitectura.md` de Renato Ponce

## PUNTOS A FAVOR

### 1. Simplicidad y claridad estructural
El documento presenta una estructura minimalista que facilita su comprensión incluso para perfiles no técnicos. Esta claridad es valiosa en entornos académicos donde participan estudiantes de diferentes niveles de experiencia. La ausencia de complejidad innecesaria permite que el equipo completo —desde desarrolladores hasta administrativos— pueda alinearse rápidamente con la visión técnica del proyecto.

### 2. Selección de herramientas maduras y probadas en la industria
Las tecnologías propuestas (GitLab, Harbor, Keycloak, Proxmox VE, Kubernetes, PostgreSQL y MinIO) representan un stack sólido con amplia adopción empresarial. Cada componente cuenta con:

- Documentación extensa y actualizada
- Comunidades activas que garantizan soporte y evolución continua
- Integraciones documentadas entre sí, lo que reduce riesgos de incompatibilidad
- Modelos de licenciamiento claros (mayoritariamente open source)

Esta elección reduce significativamente la curva de aprendizaje y los riesgos técnicos del proyecto.

### 3. Modelo híbrido bien definido
La decisión de implementar una arquitectura **on-premise + nube** es estratégicamente acertada por múltiples razones, se detallan las siguientes:
- **Soberanía de datos**: La universidad mantiene control sobre la infraestructura crítica
- **Flexibilidad operativa**: Permite escalar a la nube en momentos de alta demanda (ej. exámenes, entregas masivas)
- **Continuidad académica**: Si falla la conexión a internet, el laboratorio local sigue operativo
- **Preparación para la industria**: Los estudiantes aprenden un modelo que replica el estándar empresarial actual

### 4. Flujo de usuario final bien concebido
El flujo descrito captura el valor central del proyecto. Este enfoque:
- **Elimina la fragmentación de entornos** ("funciona en mi máquina")
- **Reduce drásticamente el tiempo de instalación** en laboratorios
- **Empodera al estudiante** para practicar desde su computadora personal
- **Estandariza la evaluación** de proyectos y prácticas

### 5. Alineación con el README.md
El archivo complementa correctamente la visión descrita en el documento principal, manteniendo coherencia en:
- La selección de tecnologías
- El enfoque híbrido
- Los objetivos de estandarización y trazabilidad

---

## PUNTOS EN CONTRA

### 1. Ausencia total de gobernanza de imágenes Docker

El documento no establece ningún mecanismo de gobierno sobre el ciclo de vida de las imágenes de contenedores, lo que genera múltiples problemas:

#### 1.1. Sin política de aprobación
- No se define **quién** autoriza la publicación de una imagen en Harbor
- No hay diferenciación entre imágenes "oficiales" (aprobadas) y "experimentales"
- Genera el riesgo que cualquier estudiante o profesor podría publicar imágenes no validadas que luego sean usadas por toda la comunidad

#### 1.2. Sin política de versionado
- No se especifica un esquema de versionado semántico (ej. `curso/version` o `proyecto/v1.2.3`)
- No se define cómo manejar versiones obsoletas o deprecadas
- Genera un riesgo de confusión con las versiones que pueden tener imágenes vulnerables

#### 1.3. Sin política de retención y obsolescencia
- No se establece un período de vida útil para las imágenes
- No se define cuándo y cómo eliminar imágenes obsoletas
- El riesgo se centra en acumular imágenes sin mantenimiento que ocupan espacio y generan confusión

---

### 2. Nula gestión de licencias de software

#### 2.1. Ignorancia de licencias en imágenes base
Las imágenes base (Ubuntu, Alpine, Debian, etc.) contienen cientos de paquetes con diferentes licencias:
- **GPLv2/v3**: Requiere compartir el código fuente si se distribuye
- **LGPL**: Permite uso comercial con restricciones de enlace dinámico
- **MIT/BSD**: Permisivas, pero requieren atribución
- **Licencias comerciales**: Pueden requerir pago si se usan en entornos productivos

El archivo no menciona cómo auditar estas licencias ni qué hacer si se detecta una incompatibilidad.

#### 2.2. Sin control de dependencias de terceros
Cuando un estudiante o profesor instala software adicional dentro de una imagen (ej. librerías Python, paquetes npm, herramientas Java), esas dependencias traen sus propias licencias. El proyecto:
- No exige un inventario de dependencias (SBOM)
- No establece un límite de licencias aceptables
- No define un proceso para aprobar nuevas dependencias

La universidad podría estar distribuyendo software con licencias incompatibles sin saberlo, exponiéndose a:
- Demandas por infracción de derechos de autor
- Obligación de liberar código propio bajo GPL
- Imposibilidad de transferir la plataforma a una empresa

#### 2.3. Sin distinción entre entorno académico y empresarial
El README.md menciona dos versiones (Académica y Empresarial), pero no refleja cómo la gestión de licencias debe ser más estricta en el entorno empresarial. Esto es un error grave porque:
- En el ámbito académico, ciertas licencias "no comerciales" son aceptables (ej. Creative Commons NC)
- En el ámbito empresarial, esas mismas licencias son prohibidas

#### 2.4. Sin seguimiento de cambios de licencia
Las licencias de software cambian con el tiempo (ej. Redis cambió su licencia en 2024). El proyecto no contempla:
- Un sistema de alertas cuando una dependencia cambia su licencia
- Un proceso para evaluar el impacto de esos cambios
- Un plan de contingencia para migrar a alternativas compatibles

---

### 3. Seguridad insuficiente en el flujo de descargas

#### 3.1. Autenticación débil o inexistente
- No se especifica cómo se autentican los estudiantes antes de descargar
- No se menciona integración con Keycloak para control de acceso
- **Riesgo**: Cualquier persona podría descargar imágenes, incluyendo competidores o actores maliciosos

#### 3.2. Sin verificación de integridad
- No se exige la verificación de checksums o firmas digitales al descargar
- **Riesgo**: Un estudiante podría descargar una imagen corrupta o modificada (ataque Man-in-the-Middle)

#### 3.3. Sin cifrado en tránsito explícito
- No se menciona el uso obligatorio de HTTPS/TLS
- **Riesgo**: Exposición de imágenes en redes no seguras (ej. WiFi público)

#### 3.4. Sin política de credenciales
- No se menciona cómo manejar credenciales dentro de las imágenes (variables de entorno, secretos)
- **Riesgo**: Imágenes que incluyen tokens, contraseñas o claves API de forma insegura

---

### 4. Omisión de monitoreo y alertas

- No se mencionan métricas de salud del sistema
- No se definen alertas para problemas comunes (ej. espacio en disco, escaneos fallidos)
- No se establece un dashboard de trazabilidad para administradores
- **Riesgo**: Problemas operativos que pasan desapercibidos hasta que afectan a los usuarios

---

### 5. Falta de documentación operativa

- No se incluyen playbooks para administradores
- No se detallan comandos, scripts o procedimientos de mantenimiento
- **Riesgo**: Dependencia crítica de conocimientos tácitos que se pierden si el personal cambia

## CONCLUSIÓN FINAL

El archivo solicitado cumple adecuadamente como un documento conceptual de alto nivel, pero es técnicamente insuficiente para guiar una implementación real debido a la omisión de aspectos críticos de gobernanza, legalidad y operación.

**Las deficiencias identificadas representan riesgos significativos que podrían:**
1. Exponer legalmente a la universidad o empresa
2. Generar pérdida de confianza en la plataforma
3. Crear problemas operativos difíciles de resolver a posteriori
4. Invalidar la transferibilidad a entornos empresariales

Por ende, se recomienda encarecidamente expandir el documento con secciones dedicadas a gobernanza, licencias, seguridad y operaciones antes de iniciar la implementación, detallando más las estructura de la arquitectura del proyecto.
