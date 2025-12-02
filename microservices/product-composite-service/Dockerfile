# === Etapa 1: La etapa de "Construcción" (Builder) ===
# Esta etapa se utiliza para desempaquetar la aplicación en capas, pero no formará parte de la imagen final.
# El objetivo es separar las dependencias (que cambian poco) del código de la aplicación (que cambia a menudo)
# para aprovechar la caché de Docker y acelerar las reconstrucciones.
FROM eclipse-temurin:24_36-jre-noble AS builder

# Establece el directorio de trabajo dentro del contenedor. Todos los comandos siguientes se ejecutarán aquí.
WORKDIR /extracted

# Copia el archivo JAR ejecutable (creado por Gradle/Maven) al contenedor y lo renombra a 'app.jar'.
ADD ./build/libs/*.jar app.jar

# ¡El paso clave de la optimización! Ejecuta la herramienta 'layertools' de Spring Boot.
# Esto no inicia la aplicación, sino que la extrae en directorios separados (capas):
# - dependencies/: Librerías de terceros.
# - spring-boot-loader/: El cargador de Spring.
# - snapshot-dependencies/: Librerías SNAPSHOT.
# - application/: Tu propio código compilado.
RUN java -Djarmode=layertools -jar app.jar extract

# === Etapa 2: La etapa "Final" ===
# Esta etapa construye la imagen final que se ejecutará. Será más ligera porque solo contiene lo necesario.
FROM eclipse-temurin:24_36-jre-noble

# Establece el directorio de trabajo final para la aplicación en ejecución.
WORKDIR /application

# Copia las capas extraídas de la etapa 'builder'. Docker puede cachear estas capas individualmente.
# Si solo cambias tu código (capa 'application'), Docker reutilizará las capas de dependencias
# de la caché, haciendo la construcción mucho más rápida.
COPY --from=builder /extracted/dependencies/ ./
COPY --from=builder /extracted/spring-boot-loader/ ./
COPY --from=builder /extracted/snapshot-dependencies/ ./
COPY --from=builder /extracted/application/ ./

# Documenta que el contenedor escuchará en el puerto 8080 en tiempo de ejecución.
# Esto coincide con la configuración del perfil 'docker' en el archivo application.yml.
EXPOSE 8080

# Define el comando que se ejecutará cuando se inicie el contenedor.
# 'JarLauncher' es una clase de Spring Boot que sabe cómo iniciar la aplicación
# a partir de las capas que hemos copiado en el directorio de trabajo.
ENTRYPOINT ["java", "org.springframework.boot.loader.launch.JarLauncher"]