<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 
                              http://maven.apache.org/xsd/settings-1.0.0.xsd">

  <servers>
    <!-- Nexus de desarrollo -->
    <server>
      <id>nexus-dev-releases</id>
      <username>usuario-dev</username>
      <password>password-dev</password>
    </server>
    <server>
      <id>nexus-dev-snapshots</id>
      <username>usuario-dev</username>
      <password>password-dev</password>
    </server>

    <!-- Nexus de producción -->
    <server>
      <id>nexus-prod-releases</id>
      <username>usuario-prod</username>
      <password>password-prod</password>
    </server>
    <server>
      <id>nexus-prod-snapshots</id>
      <username>usuario-prod</username>
      <password>password-prod</password>
    </server>
  </servers>

  <profiles>
    <!-- Perfil para despliegue a Nexus DEV -->
    <profile>
      <id>nexus-dev</id>
      <properties>
        <altDeploymentRepository>nexus-dev-releases::default::https://nexus.dev.com/repository/maven-releases/</altDeploymentRepository>
      </properties>
    </profile>

    <!-- Perfil para despliegue a Nexus PROD -->
    <profile>
      <id>nexus-prod</id>
      <properties>
        <altDeploymentRepository>nexus-prod-releases::default::https://nexus.prod.com/repository/maven-releases/</altDeploymentRepository>
      </properties>
    </profile>
  </profiles>

  <!-- No activar perfiles automáticamente -->
  <activeProfiles>
    <!-- Deja vacío para activar con -P -->
  </activeProfiles>

</settings>

