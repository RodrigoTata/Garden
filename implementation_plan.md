# Plan de Implementación: Registro Administrativo y Control de Recetas

Este plan detalla los pasos para completar el sistema de registro manual por parte de administradores y la gestión segura de credenciales y recetas médicas.

## Fase 1: Backend (Seguridad y Notificaciones) - [COMPLETADO]
- [x] **Entidad Usuario**: Añadir campos `temporaryPassword` (texto plano temporal) y `mustChangePassword`.
- [x] **Registro Administrativo**: Generar y almacenar `temporaryPassword` durante el `adminCreate`.
- [x] **Limpieza de Credenciales**: Modificar el método `update` para que al cambiar la contraseña se elimine `temporaryPassword` y se desactive `mustChangePassword`.
- [x] **Notificación Manual**: Endpoint para enviar email con la contraseña temporal real.
- [x] **Control de Recetas**: Cambiar el estado inicial de recetas procesadas a `PENDIENTE_VERIFICACION`.

## Fase 2: Frontend (UI Administrativa) - [COMPLETADO]
- [x] **Diálogo de Creación**: Formulario completo con todos los campos (RUT, Email, Apellidos, Nacimiento, Teléfono, Comuna, Ciudad).
- [x] **Confirmación de Notificación**: Implementar diálogo de confirmación antes de enviar el correo desde la lista de usuarios.
- [x] **Privacidad de Botones**: Deshabilitar el botón de "Notificar" si el usuario ya cambió su contraseña (detectado vía `canNotify`).

## Fase 3: Revisión y Pruebas (TDD) - [EN CURSO]
### Cómo realizar la revisión:
1. **Prueba de Creación**:
   - Ir a la lista de usuarios.
   - Clic en "Registrar Usuario Manual".
   - Completar todos los campos y subir al menos una receta.
   - Verificar que al finalizar se muestra la contraseña temporal en pantalla.
2. **Prueba de Receta Pendiente**:
   - Ir al perfil del usuario recién creado o a la sección de recetas.
   - Verificar que la receta aparece como **PENDIENTE** (no aprobada automáticamente).
3. **Prueba de Notificación**:
   - En la lista de usuarios, buscar al nuevo usuario.
   - Clic en el icono del sobre (Notificar).
   - Confirmar el envío en el pop-up.
   - Verificar que llega el correo con la contraseña temporal correcta.
4. **Prueba de Privacidad**:
   - Iniciar sesión como el nuevo usuario y cambiar la contraseña.
   - Volver como administrador a la lista de usuarios.
   - Verificar que el botón "Notificar" para ese usuario ahora está **deshabilitado**.

## Notas de Seguridad
> [!IMPORTANT]
> El campo `temporaryPassword` solo es visible para administradores y se elimina automáticamente en cuanto el usuario toma control de su cuenta, cumpliendo con los estándares de privacidad solicitados.
