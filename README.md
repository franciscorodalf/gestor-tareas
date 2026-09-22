# Documento de Entrega: Gestión de Versiones del JDK y Maven

---

## Fase 1. Identificar el usuario y el entorno

* **Usuario del sistema:** `fran`

---

## Fase 2. Instalar una segunda versión del JDK

En macOS, se usa el gestor de paquetes **Homebrew** para descargar JDK 17 manteniendo la versión activa (JDK 21)

```bash
# Instalación de OpenJDK 17 vía Homebrew
brew install openjdk@17

# Enlace al framework de Java de macOS para su correcta indexación
sudo ln -sfn /opt/homebrew/opt/openjdk@17/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-17.jdk
```

---

## Fase 3. Seleccionar el JDK activo

### 3.1. Entorno con JDK 17 activo

* **`JAVA_HOME` utilizado:**  
  `/opt/homebrew/Cellar/openjdk@17/17.0.20.1/libexec/openjdk.jdk/Contents/Home`

**Salida de las comprobaciones de versión en la terminal:**

```console
fran@MacBook-Air-de-Francisco gestor-tareas % java -version
openjdk version "17.0.20.1" 2026-08-18
OpenJDK Runtime Environment Homebrew (build 17.0.20.1+0)
OpenJDK 64-Bit Server VM Homebrew (build 17.0.20.1+0, mixed mode, sharing)

fran@MacBook-Air-de-Francisco gestor-tareas % javac -version
javac 17.0.20.1

fran@MacBook-Air-de-Francisco gestor-tareas % mvn -version
Apache Maven 3.9.16 (2bdd9fddda4b155ebf8000e807eb73fd829a51d5)
Maven home: /opt/homebrew/Cellar/maven/3.9.16/libexec
Java version: 17.0.20.1, vendor: Homebrew, runtime: /opt/homebrew/Cellar/openjdk@17/17.0.20.1/libexec/openjdk.jdk/Contents/Home
Default locale: es_ES, platform encoding: UTF-8
OS name: "mac os x", version: "26.6.2", arch: "aarch64", family: "mac"
```

### 3.2. Entorno con JDK 21 activo

* **`JAVA_HOME` utilizado:**  
  `/Library/Java/JavaVirtualMachines/temurin-21.jdk/Contents/Home`

**Salida de las comprobaciones de versión en la terminal:**

```console
fran@MacBook-Air-de-Francisco gestor-tareas % java -version
openjdk version "21.0.10" 2026-01-20 LTS
OpenJDK Runtime Environment Temurin-21.0.10+7 (build 21.0.10+7-LTS)
OpenJDK 64-Bit Server VM Temurin-21.0.10+7 (build 21.0.10+7-LTS, mixed mode, sharing)

fran@MacBook-Air-de-Francisco gestor-tareas % javac -version
javac 21.0.10

fran@MacBook-Air-de-Francisco gestor-tareas % mvn -version
Apache Maven 3.9.16 (2bdd9fddda4b155ebf8000e807eb73fd829a51d5)
Maven home: /opt/homebrew/Cellar/maven/3.9.16/libexec
Java version: 21.0.10, vendor: Eclipse Adoptium, runtime: /Library/Java/JavaVirtualMachines/temurin-21.jdk/Contents/Home
Default locale: es_ES, platform encoding: UTF-8
OS name: "mac os x", version: "26.6.2", arch: "aarch64", family: "mac"
```

---

## Fase 4. Construir el proyecto

Al ejecutar `mvn clean verify` con la versión **JDK 21** activa, el proyecto compila correctamente.

### Evidencia de `BUILD SUCCESS`

```text
[INFO] --- jar:3.4.2:jar (default-jar) @ gestor-tareas ---
[INFO] Building jar: /Users/fran/Documents/GitHub/gestor-tareas/target/gestor-tareas-1.0.0-SNAPSHOT.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  1.126 s
[INFO] Finished at: 2026-09-22T11:13:06+01:00
[INFO] ------------------------------------------------------------------------
```

### Identificación del archivo JAR generado

El artefacto resultante de la empaquetación se encuentra en la ruta:  
`target/gestor-tareas-1.0.0-SNAPSHOT.jar`

## Fase 5. Comprobar la relación entre JDK y Maven

### Mensaje de error capturado al construir con JDK 17

```text
[INFO] --- compiler:3.13.0:compile (default-compile) @ gestor-tareas ---
[INFO] Recompiling the module because of changed source code.
[INFO] Compiling 1 source file with javac [debug release 21] to target/classes
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.13.0:compile (default-compile) on project gestor-tareas: Fatal error compiling: error: release version 21 not supported -> [Help 1]
```

### Explicación técnica de la causa del fallo

> [!IMPORTANT]
> El archivo [`pom.xml`](file:///Users/fran/Documents/GitHub/gestor-tareas/pom.xml) del proyecto especifica la propiedad:
>
> ```xml
> <maven.compiler.release>21</maven.compiler.release>
> ```

Esta directiva obliga a Maven a requerir las especificaciones de **Java 21**. Al cambiar la versión del entorno a **JDK 17**, el plugin `maven-compiler-plugin` llama al compilador activo (`javac` v17). Como una versión anterior del JDK no posee el soporte ni las definiciones para procesar Java 21, la compilación falla lanzando un error (`release version 21 not supported`).

---

## Añadir y utilizar una dependencia

Dentro del elemento `<project>`, se incluye el bloque de dependencias:

```xml
<dependencies>
  <dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.11.0</version>
  </dependency>
</dependencies>
```

### Implementación en [`Main.java`](file:///Users/fran/Documents/GitHub/gestor-tareas/src/main/java/com/codelearn/tareas/Main.java)

Se utiliza `Gson` para convertir un mapa de datos en una cadena con formato JSON:

```java
package com.codelearn.tareas;

import com.google.gson.Gson;
import java.util.Map;

public class Main {
    public static void main(String[] args) {
        var datos = Map.of("titulo", "Aprender Maven", "completada", false);
        System.out.println(new Gson().toJson(datos));
    }
}
```

### Inspección del árbol de dependencias (`mvn dependency:tree`)

Al ejecutar `mvn dependency:tree`, Maven descarga la biblioteca y la añade al árbol de dependencias del proyecto con el scope `compile` por defecto:

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mvn dependency:tree
[INFO] Scanning for projects...
[INFO] 
[INFO] --------------------< com.codelearn:gestor-tareas >---------------------
[INFO] Building gestor-tareas 1.0.0-SNAPSHOT
[INFO]   from pom.xml
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- dependency:3.7.0:tree (default-cli) @ gestor-tareas ---
[INFO] com.codelearn:gestor-tareas:jar:1.0.0-SNAPSHOT
[INFO] \- com.google.code.gson:gson:jar:2.11.0:compile
[INFO]    \- com.google.errorprone:error_prone_annotations:jar:2.27.0:compile
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  0.687 s
[INFO] Finished at: 2026-09-22T11:35:22+01:00
[INFO] ------------------------------------------------------------------------
```

### Comprobación de errores al retirar la dependencia

Si se retira la dependencia de `pom.xml` manteniendo la importación `import com.google.gson.Gson;` en el código fuente, la compilación mediante `mvn compile` falla al no encontrar las clases.

```text
[INFO] --- compiler:3.13.0:compile (default-compile) @ gestor-tareas ---
[INFO] Recompiling the module because of changed source code.
[INFO] Compiling 1 source file with javac [debug release 21] to target/classes
[INFO] -------------------------------------------------------------
[ERROR] COMPILATION ERROR : 
[INFO] -------------------------------------------------------------
[ERROR] /Users/fran/Documents/GitHub/gestor-tareas/src/main/java/com/codelearn/tareas/Main.java:[3,23] package com.google.gson does not exist
[ERROR] /Users/fran/Documents/GitHub/gestor-tareas/src/main/java/com/codelearn/tareas/Main.java:[9,32] cannot find symbol
  symbol:   class Gson
  location: class com.codelearn.tareas.Main
[INFO] 2 errors 
[INFO] -------------------------------------------------------------
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
```

---

## Maven Central y el repositorio local

### Repositorio local y descarga de dependencias

Maven Central está disponible por defecto. La descarga de bibliotecas se almacena en el repositorio local (normalmente `~/.m2/repository` en macOS/Linux).

Comprobación de la estructura descargada para la biblioteca Gson:

```console
fran@MacBook-Air-de-Francisco gestor-tareas % ls -l ~/.m2/repository/com/google/code/gson/gson/2.11.0/
total 2304
-rw-r--r--@ 1 fran  staff     257 Sep 22 11:38 _remote.repositories
-rw-r--r--@ 1 fran  staff  637187 May 19  2024 gson-2.11.0-javadoc.jar
-rw-r--r--@ 1 fran  staff      40 Sep 22 11:38 gson-2.11.0-javadoc.jar.sha1
-rw-r--r--@ 1 fran  staff  203808 May 19  2024 gson-2.11.0-sources.jar
-rw-r--r--@ 1 fran  staff      40 Sep 22 11:38 gson-2.11.0-sources.jar.sha1
-rw-r--r--@ 1 fran  staff  298435 May 19  2024 gson-2.11.0.jar
-rw-r--r--@ 1 fran  staff      40 Sep 22 11:34 gson-2.11.0.jar.sha1
-rw-r--r--@ 1 fran  staff   11821 May 19  2024 gson-2.11.0.pom
-rw-r--r--@ 1 fran  staff      40 Sep 22 11:34 gson-2.11.0.pom.sha1
-rw-r--r--@ 1 fran  staff     167 Sep 22 11:38 m2e-lastUpdated.properties
```

### Instalación local del artefacto (`mvn install`)

Al ejecutar `mvn install`, la fase `install` del ciclo de vida empaqueta e instala el artefacto de la aplicación junto con su POM en el repositorio local:

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mvn install
[INFO] --- install:3.1.4:install (default-install) @ gestor-tareas ---
[INFO] Installing /Users/fran/Documents/GitHub/gestor-tareas/pom.xml to /Users/fran/.m2/repository/com/codelearn/gestor-tareas/1.0.0-SNAPSHOT/gestor-tareas-1.0.0-SNAPSHOT.pom
[INFO] Installing /Users/fran/Documents/GitHub/gestor-tareas/target/gestor-tareas-1.0.0-SNAPSHOT.jar to /Users/fran/.m2/repository/com/codelearn/gestor-tareas/1.0.0-SNAPSHOT/gestor-tareas-1.0.0-SNAPSHOT.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

Comprobación del contenido resultante en `~/.m2/repository/com/codelearn/gestor-tareas/1.0.0-SNAPSHOT/`:

```console
fran@MacBook-Air-de-Francisco gestor-tareas % ls -l ~/.m2/repository/com/codelearn/gestor-tareas/1.0.0-SNAPSHOT/
total 32
-rw-r--r--@ 1 fran  staff   211 Sep 22 11:53 _remote.repositories
-rw-r--r--@ 1 fran  staff  2536 Sep 22 11:51 gestor-tareas-1.0.0-SNAPSHOT.jar
-rw-r--r--@ 1 fran  staff  1272 Sep 22 11:40 gestor-tareas-1.0.0-SNAPSHOT.pom
-rw-r--r--@ 1 fran  staff   712 Sep 22 11:53 maven-metadata-local.xml
```

El repositorio local guarda tanto las descargas remotas como los artefactos propios instalados localmente.

### Empaquetado en modo offline (`mvn -o package`)

Una vez que las dependencias y plugins se han descargado al repositorio local, es posible realizar construcciones sin conexión a Internet utilizando la opción `-o` (`--offline`):

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mvn -o package
[INFO] Scanning for projects...
[INFO] --------------------< com.codelearn:gestor-tareas >---------------------
[INFO] Building gestor-tareas 1.0.0-SNAPSHOT
[INFO]   from pom.xml
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- resources:3.4.0:resources (default-resources) @ gestor-tareas ---
[INFO] skip non existing resourceDirectory /Users/fran/Documents/GitHub/gestor-tareas/src/main/resources
[INFO] 
[INFO] --- compiler:3.13.0:compile (default-compile) @ gestor-tareas ---
[INFO] Nothing to compile - all classes are up to date.
[INFO] 
[INFO] --- jar:3.4.2:jar (default-jar) @ gestor-tareas ---
[INFO] Building jar: /Users/fran/Documents/GitHub/gestor-tareas/target/gestor-tareas-1.0.0-SNAPSHOT.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

### Explicación: Publicación remota vs Instalación local

El comando `mvn install` únicamente copia el artefacto (`.jar`) y su POM al repositorio **local** de la máquina (`~/.m2/repository`), haciéndolo disponible para otros proyectos locales del mismo equipo. Para publicar el proyecto a otras personas o en servidores remotos (como Maven Central, Nexus o Artifactory), es necesario utilizar la fase `deploy` (`mvn deploy`) configurando las credenciales y el elemento `<distributionManagement>` en el [`pom.xml`](file:///Users/fran/Documents/GitHub/gestor-tareas/pom.xml).

---

## Repositorios externos y settings.xml

Separación de la configuración del proyecto ([`pom.xml`](file:///Users/fran/Documents/GitHub/gestor-tareas/pom.xml)) de la configuración del entorno (`settings.xml`).

### Configuración del entorno (`config/settings-publico.xml`)

Se crea el archivo [`config/settings-publico.xml`](file:///Users/fran/Documents/GitHub/gestor-tareas/config/settings-publico.xml) para definir un perfil de entorno que declara un repositorio externo explícito sin credenciales:

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.2.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.2.0 https://maven.apache.org/xsd/settings-1.2.0.xsd">
  <profiles>
    <profile>
      <id>repositorio-publico</id>
      <repositories>
        <repository>
          <id>central-explicito</id>
          <url>https://repo.maven.apache.org/maven2</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>false</enabled></snapshots>
        </repository>
      </repositories>
    </profile>
  </profiles>
</settings>
```

### Comprobación de perfiles activos (`-Prepositorio-publico`)

Al activar el perfil explícitamente mediante `-Prepositorio-publico`, Maven lo incluye entre los perfiles activos del proyecto:

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mvn -s config/settings-publico.xml -Prepositorio-publico help:active-profiles
[INFO] Scanning for projects...
[INFO] 
[INFO] --- help:3.5.2:active-profiles (default-cli) @ gestor-tareas ---
[INFO] 
Active Profiles for Project 'com.codelearn:gestor-tareas:jar:1.0.0-SNAPSHOT':

The following profiles are active:

 - repositorio-publico (source: external)

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

### Inspección de la configuración efectiva (`help:effective-settings`)

#### 1. Con la opción `-Prepositorio-publico`

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mvn -s config/settings-publico.xml -Prepositorio-publico help:effective-settings
...
  <activeProfiles>
    <activeProfile>repositorio-publico</activeProfile>
  </activeProfiles>
...
```

#### 2. Sin la opción `-Prepositorio-publico`

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mvn -s config/settings-publico.xml help:effective-settings
...
  <profiles>
    <profile>
      <id>repositorio-publico</id>
      ...
    </profile>
  </profiles>
  <!-- No aparece en <activeProfiles> -->
...
```

### Explicación del ejercicio: activación de perfiles y descargas desde Central

* **¿Qué cambia al ejecutar con y sin `-Prepositorio-publico`?**  
  Al incluir `-Prepositorio-publico`, el perfil se activa de forma explícita y se añade a la lista de `<activeProfiles>`, haciendo que las definiciones de repositorios dentro de ese perfil queden disponibles en la sesión de Maven. Sin la bandera `-P`, el perfil está declarado en el archivo de *settings*, pero permanece inactivo.
  
* **¿Por qué el proyecto puede seguir descargando desde Central por defecto?**  
  Porque Maven incluye implícitamente un **Super POM** base que define por defecto el repositorio **Central** (`https://repo.maven.apache.org/maven2`) con el identificador `central` para todos los proyectos Maven, sin necesidad de declararlo explícitamente en el `pom.xml` o en el `settings.xml`.

---

## Profiles: activar configuraciones de Maven

Declaración, activación y comprobación de perfiles sin duplicar el proyecto.

### Práctica guiada: distribución opcional

En el archivo [`pom.xml`](file:///Users/fran/Documents/GitHub/gestor-tareas/pom.xml), bajo `<project>`, se añadieron dos perfiles: `distribucion` (que configura `maven-shade-plugin` para empaquetar un JAR ejecutable con todas las dependencias) e `informe` (que incluye propiedades de compilación y activación por propiedad `-Dinforme=true`):

```xml
  <profiles>
    <profile>
      <id>distribucion</id>
      <build>
        <plugins>
          <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.6.0</version>
            <executions>
              <execution>
                <phase>package</phase>
                <goals><goal>shade</goal></goals>
                <configuration>
                  <shadedArtifactAttached>true</shadedArtifactAttached>
                  <shadedClassifierName>all</shadedClassifierName>
                  <createDependencyReducedPom>false</createDependencyReducedPom>
                  <transformers>
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                      <mainClass>com.codelearn.tareas.Main</mainClass>
                    </transformer>
                  </transformers>
                </configuration>
              </execution>
            </executions>
          </plugin>
        </plugins>
      </build>
    </profile>
    <profile>
      <id>informe</id>
      <activation>
        <property>
          <name>informe</name>
          <value>true</value>
        </property>
      </activation>
      <properties>
        <maven.compiler.showWarnings>true</maven.compiler.showWarnings>
      </properties>
    </profile>
  </profiles>
```

### Ejecución de los builds y verificación

#### 1. Build normal (`mvn clean package`)

Genera únicamente el JAR (`target/gestor-tareas-1.0.0-SNAPSHOT.jar`).

#### 2. Build con perfil de distribución (`mvn -Pdistribucion clean package`)

Genera además el JAR ejecutable con dependencias integradas (`target/gestor-tareas-1.0.0-SNAPSHOT-all.jar`).

#### 3. Ejecución del JAR autocontenido

```console
fran@MacBook-Air-de-Francisco gestor-tareas % java -jar target/gestor-tareas-1.0.0-SNAPSHOT-all.jar
{"titulo":"Aprender Maven","completada":false}
```

#### 4. Comprobación de perfiles activos por propiedad (`mvn -Dinforme=true help:active-profiles`)

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mvn -Dinforme=true help:active-profiles
[INFO] Scanning for projects...
[INFO] --------------------< com.codelearn:gestor-tareas >---------------------
[INFO] Building gestor-tareas 1.0.0-SNAPSHOT
[INFO]   from pom.xml
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- help:3.5.2:active-profiles (default-cli) @ gestor-tareas ---
[INFO] 
Active Profiles for Project 'com.codelearn:gestor-tareas:jar:1.0.0-SNAPSHOT':

The following profiles are active:

 - informe (source: com.codelearn:gestor-tareas:1.0.0-SNAPSHOT)

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

### Ejercicio y conceptos clave

* **Activación por propiedad (`-Dinforme=true`):** Al definir el bloque `<activation><property><name>informe</name><value>true</value></property></activation>` en el perfil `informe`, este se activa automáticamente al invocar Maven con `-Dinforme=true`, apareciendo en la salida de `help:active-profiles` sin requerir el flag `-P`.

* **Perfil de distribución optativo:** El perfil `distribucion` permanece como opcional y requiere activación explícita mediante `-Pdistribucion`.

* **Diferencia entre POM y settings:** Los perfiles del `pom.xml` modifican dependencias, plugins y la compilación/construcción (bloque `<build>`), mientras que los perfiles en `settings.xml` se limitan a propiedades y repositorios de descarga.
