# Documento de Entrega: Maven y Construcción de Proyectos Java (`gestor-tareas`)

---

## Índice de Lecciones y Pasos del Curso

1. [01. Identificar el usuario y el entorno](#01-identificar-el-usuario-y-el-entorno)
2. [02. Instalar una segunda versión del JDK](#02-instalar-una-segunda-versión-del-jdk)
3. [03. Seleccionar el JDK activo](#03-seleccionar-el-jdk-activo)
4. [04. Compilar y construir el proyecto](#04-compilar-y-construir-el-proyecto)
5. [05. Comprobar la relación entre JDK y Maven](#05-comprobar-la-relación-entre-jdk-y-maven)
6. [06. Añadir y utilizar una dependencia](#06-añadir-y-utilizar-una-dependencia)
7. [07. Maven Central y el repositorio local](#07-maven-central-y-el-repositorio-local)
8. [08. Repositorios externos y settings.xml](#08-repositorios-externos-y-settingsxml)
9. [10. Profiles: activar configuraciones de Maven](#10-profiles-activar-configuraciones-de-maven)
10. [11. Dependencias transitivas, scopes y conflictos](#11-dependencias-transitivas-scopes-y-conflictos)
11. [12. Propiedades y gestión de versiones](#12-propiedades-y-gestión-de-versiones)
12. [13. Añadir y ejecutar pruebas con JUnit](#13-añadir-y-ejecutar-pruebas-con-junit)
13. [14. El build como comprobación de calidad](#14-el-build-como-comprobación-de-calidad)
14. [15. Recursos y configuración de la aplicación](#15-recursos-y-configuración-de-la-aplicación)
15. [16. Empaquetar y ejecutar la aplicación](#16-empaquetar-y-ejecutar-la-aplicación)

---

## 01. Identificar el usuario y el entorno

* **Usuario del sistema:** `fran`
* **Directorio del proyecto:** [`gestor-tareas`](file:///Users/fran/Documents/GitHub/gestor-tareas)

---

## 02. Instalar una segunda versión del JDK

En macOS, se usa el gestor de paquetes **Homebrew** para descargar JDK 17 manteniendo la versión activa (JDK 21):

```bash
brew install openjdk@17
sudo ln -sfn /opt/homebrew/opt/openjdk@17/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-17.jdk
```

---

## 03. Seleccionar el JDK activo

### 3.1. Entorno con JDK 17 activo

* **`JAVA_HOME` utilizado:** `/opt/homebrew/Cellar/openjdk@17/17.0.20.1/libexec/openjdk.jdk/Contents/Home`

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

* **`JAVA_HOME` utilizado:** `/Library/Java/JavaVirtualMachines/temurin-21.jdk/Contents/Home`

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

## 04. Compilar y construir el proyecto

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mvn clean verify
[INFO] --- jar:3.4.2:jar (default-jar) @ gestor-tareas ---
[INFO] Building jar: /Users/fran/Documents/GitHub/gestor-tareas/target/gestor-tareas-1.0.0-SNAPSHOT.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  1.126 s
[INFO] Finished at: 2026-09-22T11:13:06+01:00
[INFO] ------------------------------------------------------------------------
```

* **Artefacto generado:** `target/gestor-tareas-1.0.0-SNAPSHOT.jar`

---

## 05. Comprobar la relación entre JDK y Maven

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mvn clean verify
[INFO] --- compiler:3.13.0:compile (default-compile) @ gestor-tareas ---
[INFO] Recompiling the module because of changed source code.
[INFO] Compiling 1 source file with javac [debug release 21] to target/classes
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.13.0:compile (default-compile) on project gestor-tareas: Fatal error compiling: error: release version 21 not supported -> [Help 1]
```

Explicación: [`pom.xml`](file:///Users/fran/Documents/GitHub/gestor-tareas/pom.xml) especifica `<maven.compiler.release>21</maven.compiler.release>`. Al compilar con JDK 17, `javac` no soporta la versión de destino Java 21, deteniendo la construcción.

---

## 06. Añadir y utilizar una dependencia

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mvn dependency:tree
[INFO] Scanning for projects...
[INFO] 
[INFO] --------------------< com.codelearn:gestor-tareas >---------------------
[INFO] Building gestor-tareas 1.0.0-SNAPSHOT
[INFO]   from pom.xml
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- dependency:3.8.1:tree (default-cli) @ gestor-tareas ---
[INFO] com.codelearn:gestor-tareas:jar:1.0.0-SNAPSHOT
[INFO] \- com.google.code.gson:gson:jar:2.11.0:compile
[INFO]    \- com.google.errorprone:error_prone_annotations:jar:2.27.0:compile
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mvn compile
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

## 07. Maven Central y el repositorio local

```console
fran@MacBook-Air-de-Francisco gestor-tareas % ls -l ~/.m2/repository/com/google/code/gson/gson/2.11.0/
total 2304
-rw-r--r--@ 1 fran  staff  637187 May 19  2024 gson-2.11.0-javadoc.jar
-rw-r--r--@ 1 fran  staff  203808 May 19  2024 gson-2.11.0-sources.jar
-rw-r--r--@ 1 fran  staff  298435 May 19  2024 gson-2.11.0.jar
-rw-r--r--@ 1 fran  staff   11821 May 19  2024 gson-2.11.0.pom
```

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mvn install
[INFO] --- install:3.1.4:install (default-install) @ gestor-tareas ---
[INFO] Installing /Users/fran/Documents/GitHub/gestor-tareas/pom.xml to /Users/fran/.m2/repository/com/codelearn/gestor-tareas/1.0.0-SNAPSHOT/gestor-tareas-1.0.0-SNAPSHOT.pom
[INFO] Installing /Users/fran/Documents/GitHub/gestor-tareas/target/gestor-tareas-1.0.0-SNAPSHOT.jar to /Users/fran/.m2/repository/com/codelearn/gestor-tareas/1.0.0-SNAPSHOT/gestor-tareas-1.0.0-SNAPSHOT.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mvn -o package
[INFO] --- compiler:3.13.0:compile (default-compile) @ gestor-tareas ---
[INFO] Nothing to compile - all classes are up to date.
[INFO] --- jar:3.4.2:jar (default-jar) @ gestor-tareas ---
[INFO] Building jar: /Users/fran/Documents/GitHub/gestor-tareas/target/gestor-tareas-1.0.0-SNAPSHOT.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

---

## 08. Repositorios externos y settings.xml

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mvn -s config/settings-publico.xml -Prepositorio-publico help:active-profiles
[INFO] --- help:3.5.2:active-profiles (default-cli) @ gestor-tareas ---
Active Profiles for Project 'com.codelearn:gestor-tareas:jar:1.0.0-SNAPSHOT':

The following profiles are active:

 - repositorio-publico (source: external)

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mvn -s config/settings-publico.xml -Prepositorio-publico help:effective-settings
...
  <activeProfiles>
    <activeProfile>repositorio-publico</activeProfile>
  </activeProfiles>
...
```

---

## 10. Profiles: activar configuraciones de Maven

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mvn clean package
[INFO] BUILD SUCCESS
```

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mvn -Pdistribucion clean package
[INFO] --- shade:3.6.0:shade (default) @ gestor-tareas ---
[INFO] Including com.google.code.gson:gson:jar:2.11.0 in the shaded jar.
[INFO] Including com.google.errorprone:error_prone_annotations:jar:2.27.0 in the shaded jar.
[INFO] Attaching shaded artifact.
[INFO] BUILD SUCCESS
```

```console
fran@MacBook-Air-de-Francisco gestor-tareas % java -jar target/gestor-tareas-1.0.0-SNAPSHOT-all.jar
{"titulo":"Aprender Maven","completada":false}
```

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mvn -Dinforme=true help:active-profiles
[INFO] --- help:3.5.2:active-profiles (default-cli) @ gestor-tareas ---
Active Profiles for Project 'com.codelearn:gestor-tareas:jar:1.0.0-SNAPSHOT':

The following profiles are active:

 - informe (source: com.codelearn:gestor-tareas:1.0.0-SNAPSHOT)

[INFO] BUILD SUCCESS
```

---

## 11. Dependencias transitivas, scopes y conflictos

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mvn dependency:tree -Dverbose -Dincludes=org.apache.commons
[INFO] com.codelearn:gestor-tareas:jar:1.0.0-SNAPSHOT
[INFO] +- org.apache.commons:commons-text:jar:1.12.0:compile
[INFO] |  \- (org.apache.commons:commons-lang3:jar:3.14.0:compile - omitted for duplicate)
[INFO] \- org.apache.commons:commons-lang3:jar:3.14.0:compile
```

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mvn dependency:tree -Dverbose -Dincludes=org.apache.commons
[INFO] com.codelearn:gestor-tareas:jar:1.0.0-SNAPSHOT
[INFO] \- org.apache.commons:commons-text:jar:1.12.0:compile
[INFO]    \- org.apache.commons:commons-lang3:jar:3.14.0:compile
```

---

## 12. Propiedades y gestión de versiones

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mvn help:effective-pom -Doutput=target/pom-efectivo.xml
[INFO] --- help:3.5.2:effective-pom (default-cli) @ gestor-tareas ---
[INFO] Effective-POM written to: /Users/fran/Documents/GitHub/gestor-tareas/target/pom-efectivo.xml
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

* **Origen de versión Gson:** Resolución de la propiedad `${gson.version}` en el bloque `<dependencyManagement>` de [`pom.xml`](file:///Users/fran/Documents/GitHub/gestor-tareas/pom.xml).
* **Origen de versiones JUnit:** Importación del BOM `org.junit:junit-bom:5.11.0` (`<scope>import</scope>`, `<type>pom</type>`).

---

## 13. Añadir y ejecutar pruebas con JUnit

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mvn test
[INFO] --- surefire:3.5.2:test (default-test) @ gestor-tareas ---
[INFO] Running com.codelearn.tareas.GestorTareasTest
[INFO] Tests run: 4, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.041 s -- in com.codelearn.tareas.GestorTareasTest
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 4, Failures: 0, Errors: 0, Skipped: 0
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mvn -Dtest=GestorTareasTest test
[INFO] Running com.codelearn.tareas.GestorTareasTest
[INFO] Tests run: 4, Failures: 0, Errors: 0, Skipped: 0
[INFO] BUILD SUCCESS
```

---

## 14. El build como comprobación de calidad

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mvn clean verify
[INFO] --- clean:3.2.0:clean (default-clean) @ gestor-tareas ---
[INFO] Deleting /Users/fran/Documents/GitHub/gestor-tareas/target
[INFO] --- compiler:3.13.0:compile (default-compile) @ gestor-tareas ---
[INFO] Compiling 2 source files with javac [debug release 21] to target/classes
[INFO] --- surefire:3.5.2:test (default-test) @ gestor-tareas ---
[INFO] Running com.codelearn.tareas.GestorTareasTest
[INFO] Tests run: 4, Failures: 0, Errors: 0, Skipped: 0
[INFO] --- jar:3.4.2:jar (default-jar) @ gestor-tareas ---
[INFO] Building jar: /Users/fran/Documents/GitHub/gestor-tareas/target/gestor-tareas-1.0.0-SNAPSHOT.jar
[INFO] BUILD SUCCESS
```

```text
[INFO] --- surefire:3.5.2:test (default-test) @ gestor-tareas ---
[INFO] Running com.codelearn.tareas.GestorTareasTest
[ERROR] Failures: 
[ERROR]   GestorTareasTest.anadeUnaTarea:13 expected: <1> but was: <0>
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
```

```console
fran@MacBook-Air-de-Francisco gestor-tareas % echo $?
0
```

---

## 15. Recursos y configuración de la aplicación

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mvn clean package
fran@MacBook-Air-de-Francisco gestor-tareas % jar tf target/gestor-tareas-1.0.0-SNAPSHOT.jar
META-INF/
META-INF/MANIFEST.MF
com/codelearn/tareas/GestorTareas.class
com/codelearn/tareas/Main.class
aplicacion.properties
```

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mvn -Pdistribucion package
fran@MacBook-Air-de-Francisco gestor-tareas % java -jar target/gestor-tareas-1.0.0-SNAPSHOT-all.jar
{"nombre":"Gestor de tareas"}
```

---

## 16. Empaquetar y ejecutar la aplicación

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mvn clean package
fran@MacBook-Air-de-Francisco gestor-tareas % jar tf target/gestor-tareas-1.0.0-SNAPSHOT.jar
fran@MacBook-Air-de-Francisco gestor-tareas % mvn dependency:copy-dependencies -DincludeScope=runtime -DoutputDirectory=target/lib
```

```console
fran@MacBook-Air-de-Francisco gestor-tareas % java -cp "target/gestor-tareas-1.0.0-SNAPSHOT.jar:target/lib/*" com.codelearn.tareas.Main
{"nombre":"Gestor de tareas"}
```

```powershell
java -cp "target/gestor-tareas-1.0.0-SNAPSHOT.jar;target/lib/*" com.codelearn.tareas.Main
```

```console
fran@MacBook-Air-de-Francisco gestor-tareas % java -jar target/gestor-tareas-1.0.0-SNAPSHOT.jar
Exception in thread "main" java.lang.NoClassDefFoundError: com/google/gson/Gson
	at com.codelearn.tareas.Main.main(Main.java:17)
Caused by: java.lang.ClassNotFoundException: com.google.gson.Gson
```

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mvn -Pdistribucion clean package
fran@MacBook-Air-de-Francisco gestor-tareas % java -jar target/gestor-tareas-1.0.0-SNAPSHOT-all.jar
{"nombre":"Gestor de tareas"}
```

### Ejercicio y verificación independiente

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mkdir -p target/prueba-ejercicio && cp target/gestor-tareas-1.0.0-SNAPSHOT.jar target/prueba-ejercicio/ && cp -r target/lib target/prueba-ejercicio/
fran@MacBook-Air-de-Francisco gestor-tareas % cd target/prueba-ejercicio && java -cp "gestor-tareas-1.0.0-SNAPSHOT.jar:lib/*" com.codelearn.tareas.Main
{"nombre":"Gestor de tareas"}
```

```console
fran@MacBook-Air-de-Francisco gestor-tareas % mkdir -p target/prueba-all && cp target/gestor-tareas-1.0.0-SNAPSHOT-all.jar target/prueba-all/
fran@MacBook-Air-de-Francisco gestor-tareas % cd target/prueba-all && java -jar gestor-tareas-1.0.0-SNAPSHOT-all.jar
{"nombre":"Gestor de tareas"}
```
