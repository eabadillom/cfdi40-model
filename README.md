# cfdi40-model

Proyecto Maven cuyo único propósito es **generar las clases Java (JAXB) a partir de los XSD del CFDI 4.0 del SAT**, para reutilizarlas como dependencia en otros proyectos.

## Alcance

Se incluyen únicamente los siguientes esquemas:

| Esquema | Archivo XSD | Paquete generado |
|---|---|---|
| CFDI 4.0 (comprobante base) | `cfdv40.xsd` | `mx.gob.sat.cfdi.v40` |
| Catálogos del SAT | `catCFDI.xsd` | `mx.gob.sat.cfdi.catalogos` |
| Tipos de datos del SAT | `tdCFDI.xsd` | `mx.gob.sat.cfdi.tipos` |
| Complemento de Pagos 2.0 | `Pagos20.xsd` | `mx.gob.sat.cfdi.pagos20` |
| Complemento de Nómina 1.2 | `nomina12.xsd` | `mx.gob.sat.cfdi.nomina12` |
| TimbreFiscalDigital 1.1 | `TimbreFiscalDigitalv11.xsd` | `mx.gob.sat.cfdi.tfd11` |

> No se incluyen otros complementos (Carta Porte, Comercio Exterior, Retenciones, etc.). Para agregarlos, ver la sección [Agregar un complemento](#agregar-un-complemento).

## Requisitos

- JDK 8 (JAXB viene incluido en el JDK; en JDK 11 o superior habría que agregar `jakarta.xml.bind-api` y una implementación de JAXB)
- Apache Maven 3.x

## Estructura del proyecto

```
cfdi40-model/
├── pom.xml
└── src/main/resources/
    ├── xsd/
    │   ├── cfdv40.xsd
    │   ├── catCFDI.xsd
    │   ├── tdCFDI.xsd
    │   ├── Pagos20.xsd
    │   ├── nomina12.xsd
    │   └── TimbreFiscalDigitalv11.xsd

    └── xjb/
        └── cfdi.xjb
```

- Los `import` dentro de los XSD apuntan a archivos **locales** (por ejemplo, `schemaLocation="catCFDI.xsd"`), por lo que la generación no requiere acceso a internet.
- El archivo `cfdi.xjb` define el paquete Java de cada XSD y mapea como `String` los catálogos demasiado grandes para ser un `enum` (`c_CodigoPostal`, `c_ClaveProdServ`, `c_ClaveUnidad`, `c_Colonia`, `c_Municipio`).

## Generación de los fuentes

```bash
mvn clean generate-sources
```

Las clases se generan en:

```
target/generated-sources/jaxb/mx/gob/sat/cfdi/...
```

Para verificar el resultado:

```bash
find target/generated-sources/jaxb -name "Comprobante.java" -o -name "Pagos.java" -o -name "Nomina.java"
```

**Notas:**

- Al final de la ejecución puede aparecer un mensaje `[INFO] ... SAXParseException: generating code` con un stack trace. Es un mensaje informativo del plugin `jaxb2-maven-plugin` 2.5.0, **no es un error**; el build termina en `BUILD SUCCESS`.
- Las clases generadas **no** se copian a `src/main/java`. Viven en `target/` y no deben versionarse; solo los `.xsd`, el `.xjb` y el `pom.xml` van a git.

## Compilación

```bash
mvn clean compile
```

## Empaquetado e instalación local

Para generar el jar con las clases compiladas e instalarlo en el repositorio local de Maven:

```bash
mvn clean install
```

Esto produce `target/cfdi40-model-1.0.0-SNAPSHOT.jar`.

## Uso desde otro proyecto

Agregar la dependencia en el `pom.xml` del proyecto consumidor:

```xml
<dependency>
  <groupId>com.ferbo.sat</groupId>
  <artifactId>cfdi40-model</artifactId>
  <version>1.0.0-SNAPSHOT</version>
</dependency>
```

Ejemplo de lectura de un CFDI con complementos:

```java
JAXBContext ctx = JAXBContext.newInstance(
    mx.gob.sat.cfdi.v40.Comprobante.class,
    mx.gob.sat.cfdi.pagos20.Pagos.class,
    mx.gob.sat.cfdi.nomina12.Nomina.class,
    mx.gob.sat.cfdi.tfd11.TimbreFiscalDigital.class
    );

Unmarshaller unmarshaller = ctx.createUnmarshaller();
Comprobante comprobante = (Comprobante) unmarshaller.unmarshal(new File("factura.xml"));
```

> Los complementos deben registrarse en el `JAXBContext` porque el nodo `cfdi:Complemento` se define con `xs:any`.

## Agregar un complemento

1. Copiar el XSD del complemento a `src/main/resources/xsd/` y ajustar sus `import` a rutas locales.
2. Agregar en `src/main/resources/xjb/cfdi.xjb` un bloque con el paquete deseado:

```xml
<jaxb:bindings schemaLocation="../xsd/NuevoComplemento.xsd" node="/xs:schema">
  <jaxb:schemaBindings>
    <jaxb:package name="mx.gob.sat.cfdi.nuevocomplemento"/>
  </jaxb:schemaBindings>
</jaxb:bindings>
```

3. Ejecutar `mvn clean generate-sources`.

## Solución de problemas

| Síntoma | Causa probable |
|---|---|
| Solo se ejecuta `maven-clean-plugin` y no se generan clases | El plugin `jaxb2-maven-plugin` está dentro de `<pluginManagement>` en lugar de `<build><plugins>` |
| Error por no encontrar un XSD o `schema_reference` a `sat.gob.mx` | Algún `import` aún apunta a una URL remota; cambiarlo a ruta local |
| Colisión de nombres entre clases | Falta el bloque del XSD en `cfdi.xjb`, o se usó `<packageName>` en el plugin (no debe usarse) |
| Error de m2e en VS Code, pero Maven funciona en terminal | El IDE usa un JDK distinto al 8; alinear `java.configuration.runtimes` o ignorar la ejecución con `lifecycle-mapping` |