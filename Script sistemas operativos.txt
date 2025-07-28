#!/bin/bash

USUARIO=$1
ROL=$2
DIR_TRUEQUE="./srv/trueque"
LOG_FILE="./gestion_usuarios-log"

log_action() {
echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" >> "$LOG_FILE"
}

mostrar_uso() {
echo "Uso: $0 <usuario> <rol: admin|user>"
exit 1
}

echo "========================================================"
echo " GESTION DE USUARIOS"
echo "========================================================"

if [ $# -ne 2 ]; then
echo "Error: Número incorrecto de parámetros"
mostrar_uso
fi

if [ "$ROL" != "admin" ] && [ "$ROL" != "user" ]; then
echo "Error: Rol inválido. Use 'admin' o 'user'"
log_action "ERROR: Intento de crear usuario $USUARIO con rol inválido: $ROL"
mostrar_uso
fi

mkdir -p "$(dirname "$LOG_FILE")"
log_action "INICIO: Procesando usuario $USUARIO con rol $ROL"

USER_FILE="./users_db/$USUARIO.txt"
mkdir -p "./users_db"

if [ -f "$USER_FILE" ]; then
echo "El usuario '$USUARIO' ya existe"
log_action "INFO: Usuario $USUARIO ya existe"
EXISTING_ROL=$(cat "$USER_FILE")
echo "Rol actual: $EXISTING_ROL"
else
echo "Creando usuario '$USUARIO'"
echo "$ROL" > "$USER_FILE"
log_action "CREADO: Usuario $USUARIO con rol $ROL"
fi

GROUP_FILE="./groups_db/$ROL.txt"
mkdir -p "./groups_db"

if [ ! -f "$GROUP_FILE" ]; then
echo "Creando grupo '$ROL'"
echo "$(date)" > "$GROUP_FILE"
log_action "CREADO: Grupo $ROL"
fi

echo "$USUARIO" >> "$GROUP_FILE"
log_action "AGREGADO: Usuario $USUARIO al grupo $ROL"

if [ ! -d "$DIR_TRUEQUE" ]; then
echo "Creando directorio '$DIR_TRUEQUE'"
mkdir -p "$DIR_TRUEQUE"
log_action "CREADO: Directorio $DIR_TRUEQUE"
fi

PERMISOS_FILE="$DIR_TRUEQUE/.permiso"

if [ "$ROL" = "admin" ]; then
echo "Asignando permiso rwx para grupo admin"
echo "rwxrwx---:admin:$USUARIO:$(date '+%Y-%m-%d')" > "$PERMISOS_FILE"
chmod 770 "$DIR_TRUEQUE" 2>/dev/null || true
log_action "PERMISOS: rwx asignados a admin"
else
echo "Asignando permisos rx para grupo user"
echo "rwxr-x---:user:$USUARIO:$(date '+%Y-%m-%d')" > "$PERMISOS_FILE"
chmod 750 "$DIR_TRUEQUE" 2>/dev/null || true
log_action "PERMISOS: rx asignados a user"
fi

echo ""
echo "Resumen:"
echo "Usuario: $USUARIO"
echo "Rol: $ROL"
echo "Directorio: $DIR_TRUEQUE"
echo "Log: $LOG_FILE"
echo ""
echo "Permisos actuales:"
cat "$PERMISOS_FILE"