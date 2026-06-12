# Guía de Configuración - GitHub Movinex

## Primer Push: Clonar y Enviar Cambios

### 1. Clonar el Repositorio

```bash
git clone https://github.com/emagrassi-arch/movinex.git
cd movinex
```

### 2. Configurar Git (si aún no lo has hecho)

```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tu-email@ejemplo.com"
```

### 3. Ver el Estado

```bash
git status
```

### 4. Agregar Cambios

Para agregar todos los cambios:
```bash
git add .
```

Para agregar archivos específicos:
```bash
git add nombre_del_archivo
```

### 5. Crear un Commit

```bash
git commit -m "Mensaje descriptivo del cambio"
```

Ejemplos:
- `git commit -m "feat: Agregar cotizador.html"`
- `git commit -m "docs: Actualizar FAQ.md"`
- `git commit -m "fix: Corregir cálculo de CAT"`

### 6. Enviar al Repositorio Remoto

```bash
git push origin main
```

O si es la primera vez con una rama:
```bash
git push -u origin main
```

## Autenticación en GitHub

### Opción A: HTTPS + Personal Access Token (Recomendado)

1. Ve a GitHub Settings → Developer settings → Personal access tokens
2. Crea un token con permisos `repo` y `workflow`
3. Usa el token como contraseña cuando git lo pida

### Opción B: SSH (Más seguro a largo plazo)

Genera las claves SSH:
```bash
ssh-keygen -t ed25519 -C "tu-email@ejemplo.com"
```

Agrega la clave pública a tu perfil de GitHub:
Settings → SSH and GPG keys → New SSH key

Cambia la URL del repo:
```bash
git remote set-url origin git@github.com:emagrassi-arch/movinex.git
```

## Flujo de Trabajo Recomendado

```
1. git status          # Ver cambios
2. git add .           # Preparar cambios
3. git commit -m "..."  # Registrar cambios
4. git push            # Enviar a GitHub
5. git pull            # Obtener cambios remotos (antes de trabajar)
```

## Resolver Conflictos

Si `git push` falla por conflictos:

```bash
git pull origin main
# Resuelve los conflictos manualmente en los archivos
git add .
git commit -m "Resolver conflictos"
git push origin main
```

## Preguntas Frecuentes

**P: ¿Cómo veo el historial de commits?**
```bash
git log
git log --oneline
```

**P: ¿Cómo revierto un cambio?**
```bash
git revert HASH_DEL_COMMIT
git checkout archivo.txt  # Revertir un archivo específico
```

**P: ¿Cómo borro una rama local?**
```bash
git branch -d nombre_rama
```

---

Más info: https://git-scm.com/doc
