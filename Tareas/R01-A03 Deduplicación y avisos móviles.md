---
estado: libre
responsable: Alex (PC 2)
pc: PC 2
depende_de: []
rama:
---
# R01-A03 Deduplicación y avisos móviles

Origen: [[R01 Revisión de G06 y G09 (unidas sin revisar)]], hallazgo Codex [P2].

Evidencia: mobile/src/hooks/useAvisosOfertas.js:28-35 lee vistos del shim síncrono de localStorage; mobile/src/services/ubicacionFondo.js:52-56 escribe directamente AsyncStorage. mobile/src/lib/storage.js solo hidrata su memoria una vez. Lo avisado en background puede no existir en la copia en memoria al volver y volver a generar notificación y POST. La comprobación AppState al inicio de la tarea no evita carreras durante awaits.

También se marcan/contabilizan todos los nuevos antes de notificar: sin permiso, notificarLocal retorna sin mostrar nada; si hay más de tres, los que no caben en la lista se cuentan sin mostrarse. En web hay fix separado fix/R01-avisos-visibles (43b24f0).

Alex: usar una fuente coherente y serializar reclamación por restaurante/nivel/día entre foreground y background; registrar únicamente avisos efectivamente mostrados. Decidir cómo reconciliar fallos parciales sin inventar garantía exactly-once del POST (el backend es anónimo y no deduplica por usuario). Coordinar con C02/Claude para consentimiento, permisos y privacidad, sin guardar posiciones.

Terminada cuando: mismo restaurante/nivel no se repite al pasar foreground↔background, tests con operaciones concurrentes, permiso de notificación denegado y más de tres candidatos. Un nivel mejor sí se muestra. Prueba física de Android.
