BCS – PyPSA‑SPICE | global_input (MX) 
====================================================================

Este paquete contiene archivos de **global_input** necesarios para que PyPSA‑SPICE lea perfiles horarios y parámetros
de tecnologías para México (MX), usados aquí para Baja California Sur (BCS).
**Referencia / Línea Base (LB)** 

----------------------------------------------------------------------------------
1) Fuentes 
----------------------------------------------------------------------------------

A)  **PROFILES_CEM_BCS_LB 1.csv**
  - Usado para construir perfiles en:
    - demand_profile.csv (demanda, normalizada a energía anual)
    - availability.csv (renovables, capacity factors 0–1)

B) Información del template de PyPSA‑SPICE (placeholders necesarios para correr)
- technologies.csv, power_plant_costs.csv, storage_costs.csv (y otros global_input)
  - Se copiaron parámetros/costos del país “XY” del template hacia “MX” para evitar faltantes.
  - **Estos costos no deben interpretarse como valores oficiales para México**: son placeholders para validación de pipeline.

----------------------------------------------------------------------------------
2) Archivos clave y cómo se usan
----------------------------------------------------------------------------------

2.1 demand_profile.csv
- Contiene perfiles horarios **normalizados a energía** (la suma anual = 1) por nodo:
  - EDEM.EDEM_N50 / N51 / N52 / N53  (LB – Referencia)


2.2 availability.csv
- Contiene **disponibilidad 0–1** (capacity factor / p_max_pu) por nodo y tecnología:
  - PHOT (PV utility): PV.PV_N50..N53
  - RTPV (PV distribuida): PVGD_LB_N50..N53 (si no existe N53 en datos, puede quedar 0 o proxy)
  - WTON (viento onshore): WIND.WIND_N50 y WIND.WIND_N51 (otros nodos quedan 0)

2.3 renewables_technical_potential.csv
- Se define un “techo” de potencial técnico (MW) por tecnología/nodo.
- En esta versión, el potencial PHOT se alineó a los valores 2035 del escenario LB (PV.*.PMAX de José).

----------------------------------------------------------------------------------
3) Consideraciones prácticas
----------------------------------------------------------------------------------
- Longitud de series: los perfiles son de **8760 horas** (año no bisiesto). Si se corre un año bisiesto (8784),
  debe ajustarse el set de snapshots o extender perfiles.
- Para correr un escenario alterno, bastaría cambiar `profile_type` en loadss.csv para apuntar a `EDEM_ICM.*`, ojo este ahorita no se considera 
  si esos perfiles están incluidos despues, para la validación actual, se usa LB.
----------------------------------------------------------------------------------
Actualización de costos tecnológicos (CAPEX/OPEX/VOM)
----------------------------------------------------------------------------------
Se actualizaron los archivos:
- power_plant_costs.csv
- storage_costs.csv

Fuente:
- CAPEX_OPEX_VOM_NDC_ICM.xlsx (hoja: Costos_NDC_2024)

Conversión de unidades:
- CAPEX [USD/kW]  -> cap__usd_mw [USD/MW] multiplicando por 1000
- OPEX  [USD/kW]  -> fom__usd_mwa [USD/MW-a] multiplicando por 1000 (asumido anual)
- VOM   [USD/MWh] -> vom__usd_mwh [USD/MWh] sin cambio

Baterías:
- Componente potencia (inversor) se asigna en power_plant_costs.csv para BATS usando “Bateria USD per kW”.
- Componente energía se asigna en storage_costs.csv para BATS usando “Bateria USD per kWh”.
- FOM energía: se convierte de OPEX [USD/kW-a] a [USD/MWh-a] asumiendo 4h (1 kW ↔ 4 kWh).

----------------------------------------------------------------------------------
Ajuste de costos: RTPV = PHOT (MX)
----------------------------------------------------------------------------------
Para evitar el uso de costos "default" en PV distribuida, se alinearon los costos de RTPV con PHOT
para MX en años 2025/2030/2035 dentro de power_plant_costs.csv.
ahora se considera RTPV (rooftop/distribuida) = PHOT (utility-scale) en CAPEX/FOM/VOM/life.
