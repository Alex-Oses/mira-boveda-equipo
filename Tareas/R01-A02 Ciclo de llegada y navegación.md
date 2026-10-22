---
estado: libre
responsable: Alex (PC 2)
pc: PC 2
depende_de: []
rama:
---
# R01-A02 Ciclo de llegada y navegación

Origen: [[R01 Revisión de G06 y G09 (unidas sin revisar)]], hallazgo Codex [P2].

Evidencia: ambos hooks useLlegada.js:27-49 usan pedidaAuto para toda la vida de la instancia, sin depender del UID; el POST no tiene generación/guardia. Cambiar restaurante en la misma instancia no dispara creación nueva; un POST viejo puede sustituir el QR nuevo. Un cambio de cuenta no retira inmediatamente el QR. En móvil no se comprueba foco ni AppState; las rutas pueden quedar montadas al navegar.

Alex: ligar visita y POST al UID/restaurante vigente, invalidar resultados antiguos y reiniciar la creación al cambiar contexto. Sondear solo con pantalla activa/visible, consentimiento vigente y sesión válida; al salir/cambiar de cuenta retirar QR. Coordinar C02 y revisión visual de Claude. No ampliar el contrato.

El fix local fix/R01-sondeo-visitas (4633d80) resuelve solapamiento y respuestas GET tras limpieza, y 401/403/404 terminales; revisar esta tarea sobre main cuando se haya integrado, sin duplicar ese fix.

Terminada cuando: pruebas con cambio A→B y restaurante sin desmontar, POST lento al salir, blur/foco y app a background/foreground; no se muestran QR de otro contexto ni se sondea una pantalla oculta.
