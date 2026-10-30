# Guía de uso en Windows 11 + Docker Desktop + Jenkins

## Qué se corrigió
- Jenkinsfile vacío reemplazado por pipeline de cinco etapas.
- Escape HTML restaurado en `/hello` para evitar XSS reflejado.
- Se conserva Pipfile.lock del docente para una instalación reproducible.
- `requirements.txt` antiguo se deja como nota para evitar instalar versiones incompatibles.
- Dockerfile.jenkins instala Python, pip y pipenv de manera persistente en una imagen personalizada.
- Pruebas con pytest, análisis con flake8/bandit y auditoría de dependencias con pip-audit.

## IMPORTANTE: usar el Jenkins existente sin perder datos
El volumen `jenkins_home` guarda la configuración. Antes de cambiar el contenedor, **comprueba** con `docker inspect jenkins` que realmente utiliza el volumen `jenkins_home`.

En PowerShell, dentro de esta carpeta:

```powershell
docker build -f Dockerfile.jenkins -t jenkins-python-pipenv .
docker stop jenkins
docker rename jenkins jenkins-anterior
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home --restart unless-stopped jenkins-python-pipenv
```

El contenedor anterior se conserva como respaldo y queda detenido. Si el puerto 8080 está ocupado, revisa `docker ps -a`.

## Jenkins
1. Publica este proyecto en un repositorio GitHub (puede ser uno nuevo).
2. Jenkins > New Item > Pipeline > Pipeline script from SCM > Git.
3. Introduce URL del repositorio, rama `*/main` y ruta `Jenkinsfile`.
4. Ejecuta Build Now y revisa Console Output.
5. Guarda capturas de las cinco etapas, pruebas y auditoría.

## Consideraciones
- La etapa Dependency Management instala exactamente el lock del profesor, **no actualiza automáticamente** versiones: las actualizaciones requieren revisar y regenerar el lock.
- La etapa Analyze puede fallar si pip-audit detecta vulnerabilidades; es el comportamiento deseado para impedir desplegar dependencias conocidas como vulnerables.
- El deploy es una simulación local de prueba, no un despliegue público.
- La instalación depende de conexión a PyPI desde el contenedor.
- No se ha ejecutado Jenkins/Docker desde este entorno: valida el pipeline en tu equipo.
