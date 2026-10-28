# Changelog

Todos los cambios importantes en este proyecto serán documentados en este archivo.

Este proyecto sigue versionado semántico (SemVer) y el formato de [Keep a Changelog](https://keepachangelog.com/es-ES/1.0.0/).

## [1.0.0] - 2026-10-26

### Added

- Proyecto Maven `cfdi40-model` para generar clases Java (JAXB) a partir de los XSD del CFDI 4.0 del SAT, compatible con JDK 8.
- Configuración del plugin `jaxb2-maven-plugin` 2.5.0 para ejecutar `xjc` en la fase `generate-sources`, con salida en `target/generated-sources/jaxb`.
- Esquemas incluidos en `src/main/resources/xsd/`:
  - CFDI 4.0 (`cfdv40.xsd`).
  - Catálogos del SAT (`catCFDI.xsd`).
  - Tipos de datos del SAT (`tdCFDI.xsd`).
  - Complemento de Pagos 2.0 (`Pagos20.xsd`).
  - Complemento de Nómina 1.2 (`nomina12.xsd`).
  - Timbre Fiscal Digital 1.1 (`TimbreFiscalDigitalv11.xsd`).
- Archivo de bindings `cfdi.xjb` que asigna un paquete Java por esquema:
  - `mx.gob.sat.cfdi.v40`
  - `mx.gob.sat.cfdi.catalogos`
  - `mx.gob.sat.cfdi.tipos`
  - `mx.gob.sat.cfdi.pagos20`
  - `mx.gob.sat.cfdi.nomina12`
  - `mx.gob.sat.cfdi.tfd11`
- Mapeo a `String` de los catálogos demasiado grandes para ser `enum` (`c_CodigoPostal`, `c_ClaveProdServ`, `c_ClaveUnidad`, `c_Colonia`, `c_Municipio`), lo que elimina los avisos de `EnumMemberSizeCap` durante la generación.
- `import` de los XSD apuntando a archivos locales, de modo que la generación no requiere acceso a internet.
- `distributionManagement` para publicar en GitHub Packages.
- Flujo de integración continua en GitHub Actions (`.github/workflows/deploy-snapshot.yml`) que valida la versión SNAPSHOT, genera las fuentes, verifica las clases generadas (`Comprobante`, `Pagos`, `Nomina`, `TimbreFiscalDigital`), compila y publica en GitHub Packages desde la rama `develop`.
- Flujo de integración continua en GitHub Actions (`.github/workflows/deploy-release.yml`) que valida la versión RELEASE, genera las fuentes, verifica las clases generadas (`Comprobante`, `Pagos`, `Nomina`, `TimbreFiscalDigital`), compila y publica en GitHub Packages desde la rama `master`.
- Documentación en `README.md`: generación de fuentes, compilación, instalación local, uso desde otros proyectos, cómo agregar complementos y solución de problemas.
