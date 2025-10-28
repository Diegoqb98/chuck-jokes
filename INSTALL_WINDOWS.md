# 🪟 Guía de Instalación en Windows

Esta guía detalla paso a paso cómo instalar y configurar el proyecto Chuck Jokes en Windows usando XAMPP.

## 📋 Tabla de Contenidos

1. [Requisitos Previos](#requisitos-previos)
2. [Instalación de XAMPP](#instalación-de-xampp)
3. [Instalación de Composer](#instalación-de-composer)
4. [Configuración del Proyecto](#configuración-del-proyecto)
5. [Solución de Problemas](#solución-de-problemas)

## 🔧 Requisitos Previos

- Windows 10 o superior
- Conexión a Internet
- Permisos de administrador
- ~500 MB de espacio en disco

## 📦 Instalación de XAMPP

### Paso 1: Descargar XAMPP

1. Visita [https://www.apachefriends.org/](https://www.apachefriends.org/)
2. Descarga **XAMPP 8.2.12** o superior (incluye PHP 8.2+)
3. Ejecuta el instalador descargado

### Paso 2: Instalar XAMPP

1. Acepta los componentes por defecto (Apache, MySQL, PHP, phpMyAdmin)
2. Instala en `C:\xampp` (ruta recomendada)
3. Desmarca "Start the Control Panel now" si no quieres Apache automáticamente
4. Completa la instalación

### Paso 3: Configurar PHP

1. Abre el archivo `C:\xampp\php\php.ini` con el Bloc de notas (ejecutado como administrador)
2. Busca las siguientes líneas (usa `Ctrl+F` para buscar):

```ini
;extension=intl
;extension=mbstring
;extension=pdo_sqlite
;extension=curl
;extension=openssl
```

3. Elimina el punto y coma (`;`) al inicio de cada línea:

```ini
extension=intl
extension=mbstring
extension=pdo_sqlite
extension=curl
extension=openssl
```

4. Guarda el archivo (`Ctrl+S`)
5. Cierra el Bloc de notas

## 📦 Instalación de Composer

### Método 1: Instalador de Windows (Recomendado)

1. Visita [https://getcomposer.org/download/](https://getcomposer.org/download/)
2. Descarga `Composer-Setup.exe`
3. Ejecuta el instalador
4. Cuando te pregunte por PHP, selecciona: `C:\xampp\php\php.exe`
5. Deja las opciones por defecto y completa la instalación
6. Abre PowerShell y verifica la instalación:

```powershell
composer --version
```

Deberías ver algo como: `Composer version 2.8.12`

### Método 2: Instalación Manual

```powershell
# Descargar Composer
Invoke-WebRequest -Uri "https://getcomposer.org/installer" -OutFile "C:\xampp\composer-setup.php"

# Instalar Composer
php C:\xampp\composer-setup.php --install-dir=C:\xampp\php

# Verificar instalación
php C:\xampp\php\composer.phar --version
```

## 🚀 Configuración del Proyecto

### Paso 1: Clonar el Repositorio

```powershell
# Abrir PowerShell como Administrador
# Navegar a la carpeta htdocs
cd C:\xampp\htdocs

# Clonar el proyecto
git clone https://github.com/maximofernandezriera/chuck-jokes.git

# Entrar al directorio
cd chuck-jokes
```

### Paso 2: Instalar Dependencias

```powershell
# Instalar paquetes de Composer
composer install --no-dev

# Si hay problemas con la memoria, aumenta el límite:
php -d memory_limit=-1 C:\xampp\php\composer.phar install --no-dev
```

### Paso 3: Configurar la Base de Datos

```powershell
# Crear el directorio tmp si no existe
New-Item -ItemType Directory -Force -Path tmp

# Crear el archivo de base de datos SQLite
New-Item -ItemType File -Force -Path tmp\database.sqlite
```

### Paso 4: Crear Configuración Local

Crea el archivo `config\app_local.php` con este contenido:

```php
<?php
return [
    'Datasources' => [
        'default' => [
            'driver' => Cake\Database\Driver\Sqlite::class,
            'database' => ROOT . DS . 'tmp' . DS . 'database.sqlite',
            'url' => env('DATABASE_URL', null),
        ],
        'test' => [
            'driver' => Cake\Database\Driver\Sqlite::class,
            'database' => ROOT . DS . 'tmp' . DS . 'test.sqlite',
        ],
    ],
    
    'DebugKit' => [
        'ignoreAuthorization' => true,
    ],
];
```

### Paso 5: Ejecutar Migraciones

```powershell
# Crear las tablas de la base de datos
php bin\cake.php migrations migrate

# Limpiar caché
php bin\cake.php cache clear_all
```

### Paso 6: Iniciar el Servidor

```powershell
# Iniciar el servidor PHP integrado
php -S localhost:8765 -t webroot
```

Verás un mensaje como:
```
[Mon Oct 28 21:00:00 2025] PHP 8.2.12 Development Server (http://localhost:8765) started
```

### Paso 7: Probar la Aplicación

Abre tu navegador y visita:

- **Chiste aleatorio**: [http://localhost:8765/jokes/random](http://localhost:8765/jokes/random)
- **Chistes guardados**: [http://localhost:8765/jokes/index](http://localhost:8765/jokes/index)

## 🚨 Solución de Problemas

### Error: "strict_types declaration must be the very first statement"

**Causa**: El archivo tiene un BOM (Byte Order Mark) UTF-8 invisible.

**Solución**:

```powershell
# Navegar al proyecto
cd C:\xampp\htdocs\chuck-jokes

# Eliminar el BOM del controlador
$content = Get-Content -Path 'src\Controller\JokesController.php' -Raw
[System.IO.File]::WriteAllText('src\Controller\JokesController.php', $content, [System.Text.UTF8Encoding]::new($false))

# Refrescar la página en el navegador
```

### Error: "Class 'IntlDateFormatter' not found"

**Causa**: La extensión `intl` no está habilitada.

**Solución**:
1. Edita `C:\xampp\php\php.ini`
2. Busca `;extension=intl`
3. Elimina el `;` para dejarlo como `extension=intl`
4. Reinicia el servidor PHP

### Error: "Unable to write to tmp directory"

**Causa**: Permisos insuficientes en la carpeta `tmp`.

**Solución**:

```powershell
# Dar permisos completos a la carpeta tmp
icacls "C:\xampp\htdocs\chuck-jokes\tmp" /grant Everyone:F /T

# O crear de nuevo con permisos
Remove-Item -Path tmp -Recurse -Force
New-Item -ItemType Directory -Force -Path tmp
New-Item -ItemType File -Force -Path tmp\database.sqlite
```

### Error: "Port 8765 is already in use"

**Causa**: Otro proceso está usando el puerto 8765.

**Solución 1** - Usar otro puerto:
```powershell
php -S localhost:8770 -t webroot
```

**Solución 2** - Encontrar y detener el proceso:
```powershell
# Encontrar el proceso
Get-Process -Id (Get-NetTCPConnection -LocalPort 8765).OwningProcess

# Detener el proceso (reemplaza PID con el número que aparece)
Stop-Process -Id PID -Force
```

### Error: "SQLSTATE[HY000]: General error: 8 attempt to write a readonly database"

**Causa**: La base de datos SQLite no tiene permisos de escritura.

**Solución**:

```powershell
# Dar permisos de escritura al archivo de base de datos
icacls "C:\xampp\htdocs\chuck-jokes\tmp\database.sqlite" /grant Everyone:F

# Y a la carpeta tmp también
icacls "C:\xampp\htdocs\chuck-jokes\tmp" /grant Everyone:F /T
```

### Apache no inicia / ERR_CONNECTION_REFUSED

**Causa**: Puerto 80 ocupado o conflicto con otro servicio.

**Solución**: Usar el servidor PHP integrado en su lugar (puerto 8765):

```powershell
cd C:\xampp\htdocs\chuck-jokes
php -S localhost:8765 -t webroot
```

### Composer es muy lento

**Solución**: Aumentar el timeout de Composer:

```powershell
composer config --global process-timeout 2000
composer install --no-dev
```

## 🔍 Comandos Útiles

### Verificar instalación de PHP:
```powershell
php -v
php -m  # Ver extensiones cargadas
```

### Verificar instalación de Composer:
```powershell
composer --version
composer diagnose  # Diagnosticar problemas
```

### Ver procesos en un puerto:
```powershell
Get-NetTCPConnection -LocalPort 8765
```

### Limpiar caché de CakePHP:
```powershell
php bin\cake.php cache clear_all
php bin\cake.php schema_cache clear
```

### Consultar la base de datos SQLite:
```powershell
# Instalar SQLite primero si no lo tienes
# Descargar de: https://www.sqlite.org/download.html

sqlite3 tmp\database.sqlite "SELECT * FROM jokes;"
```

## 📚 Referencias

- [Documentación de CakePHP 5](https://book.cakephp.org/5/en/index.html)
- [Documentación de XAMPP](https://www.apachefriends.org/faq_windows.html)
- [Documentación de Composer](https://getcomposer.org/doc/)
- [Chuck Norris API](https://api.chucknorris.io/)

---

**¿Necesitas ayuda?** Abre un issue en: [https://github.com/maximofernandezriera/chuck-jokes/issues](https://github.com/maximofernandezriera/chuck-jokes/issues)
