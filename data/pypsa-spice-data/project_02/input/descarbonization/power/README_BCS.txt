PyPSA‑SPICE | BCS (Baja California Sur) 
========================================================

Este folder contiene las **entradas del sector eléctrico (power)** para representar el sistema de Baja California Sur (BCS)
en PyPSA‑SPICE. Los archivos se organizan para describir: (i) nodos/buses eléctricos, (ii) demanda, (iii) generadores y
tecnologías, (iv) red de transmisión (interconectores), (v) combustibles y emisiones, y (vi) almacenamiento.

Escenario modelado: Referencia (Línea Base)
-------------------------------------------
El **Escenario de Referencia** representa la trayectoria base del sistema eléctrico de Baja California Sur, construida a partir
de la planeación y supuestos disponibles hasta mediados de 2025. De forma general:

- En 2025–2030, el sistema incorpora la **nueva central de combustión interna (CCI) en Los Cabos** contemplada en la planeación oficial vigente
  al momento de la configuración del escenario.
- No incluye centrales termosolares anunciadas posteriormente (por no estar consideradas en la modelación original).

Notas importantes de implementación en PyPSA‑SPICE
-------------------------------------------------
- Los archivos de esta carpeta se complementan con `global_input/`, donde viven **perfiles horarios** (demanda y disponibilidad renovable) y parámetros
  comunes de tecnologías.
- En PyPSA‑SPICE, los combustibles (gas/oil) se representan con **buses de commodity** (por ejemplo `MX_GASN`, `MX_OILN`) que no son nodos geográficos:
  sirven para enlazar plantas térmicas con costos/abastecimiento de combustible.
- Los nombres de nodos eléctricos usados en este caso son:
  - N50_VILLA_CONSTITUCION
  - N51_LA_PAZ
  - N52_LOS_CABOS
  - N53_MULEGE

Fuentes de datos 
-------------------------------
Este paquete integra información técnica, económica y operativa proveniente de fuentes oficiales y de bases especializadas, con el propósito de representar
con precisión la estructura eléctrica de Baja California Sur. En esta versión, las fuentes concretas utilizadas para poblar los CSV son:

1) Configuración LB (2030 y 2035) por nodo/tecnología/unidad:
   - `Resumen Configuracion BCS 1.xlsx` (hojas `LB_30` y `LB_35`) 
     Incluye, entre otros: demanda pico nodal (`EDEM.*.PSET`), capacidades (`PV.*.PMAX`, `WIND.*.PMAX`, `PVGD.*.PSET`),
     almacenamiento (`ESTR.*.PGMAX` y `ESTR.*.MAXCAP`), límites de transmisión (`LI.*.PMAX`), y precios de combustibles (`FUEL.*.FuelPrice` en $/GJ).

2) Perfiles horarios (forma temporal):
   - `PROFILES_CEM_BCS_LB 1.csv`
     Contiene series horarias normalizadas para demanda y renovables (8760 horas). En esta carpeta se usan vía `global_input/`.

3) Parámetros/costos tecnológicos generales:
   - En esta etapa de validación, algunos parámetros/costos pueden provenir de plantillas del repositorio (template) hasta que se incorporen supuestos “oficiales”
     (por ejemplo IRENA/NREL/SENER) en `global_input/`.

Descripción por archivos 
-----------------------------------------------

1) `buses.csv`
- Define buses del modelo.
- Incluye buses eléctricos por nodo (por ejemplo `N51_LA_PAZ_HVELEC`) y buses de commodity (por ejemplo `MX_GASN`, `MX_OILN`, `MX_ATMP`).
- Estos últimos permiten modelar consumo de combustibles y contabilidad de emisiones.

2) `loadss.csv`
- Demanda anual por nodo y año (`total_load__mwh`) y la referencia al perfil horario (`profile_type`).
- Para 2030 y 2035, la demanda anual por nodo se calcula usando:
  - pico nodal (MW) del `Resumen Configuracion BCS` (`EDEM.EDEM_N*.PSET`) y
  - el perfil horario LB (`EDEM.EDEM_N*` del archivo de perfiles),
  con la relación:  total_load__mwh = PSET_peak_MW × Σ_t perfil(t).
- El perfil horario se encuentra en `global_input/demand_profile.csv`.

3) `power_generators.csv`
- Generadores no térmicos (principalmente renovables y geotermia) por nodo.
- Campos típicos: tipo de tecnología (ej. `PHOT` PV utility, `RTPV` PV distribuida, `WTON` eólica, `GEOT` geotermia),
  límites de capacidad por año  y bus eléctrico asociado.

4) `power_links.csv`
- Conversiones “link” para tecnologías que consumen un insumo y producen electricidad (por ejemplo térmicas).
- En este modelo: links de combustible → electricidad (bus0 = combustible; bus1 = electricidad del nodo).

5) `interconnector.csv`
- Enlaces eléctricos entre nodos (capacidad de transmisión).
- Para 2030 y 2035 se fijan límites con `LI.*.PMAX` del `Resumen Configuracion BCS`.

6) `fuel_suppliess.csv`
- Define el costo y/o límite de abastecimiento anual de combustible para buses de commodity.
- Unidades:
  - `fuel_cost__usd_mwh` está en USD por MWh de energía del combustible.
  - Si la fuente viene en USD/GJ, se convierte con 1 MWh = 3.6 GJ.
- En esta versión se usa `MX_GASN` para gas natural y `MX_OILN` como bus agregado de líquidos (diesel/combustóleo) si no hay separación explícita.

7) `storage_capacity.csv`
- Potencia instalada (MW) de almacenamiento por nodo (baterías) y límites por año.

8) `storage_energy.csv`
- Energía almacenada máxima (MWh) por unidad de almacenamiento y por año.

Cómo correr el caso
-------------------
1) Colocar este folder `power/` en el directorio de inputs del escenario correspondiente (por ejemplo `project_02/input/scenario_01/power/`).
2) Asegurar que `global_input/` (ver README correspondiente) esté disponible y contenga los perfiles horarios.
3) Ejecutar el pipeline de PyPSA‑SPICE para construir/redespachar la red.

pendienes
---------------------------------
- Si se requiere replicar con total fidelidad el despacho térmico por combustible, hace falta el mapeo completo de unidades térmicas (FGEN) a nodo y combustible.
- Si se desea pasar de “validación de referencia” a “optimización con expansión”, ajustar `p_nom_extendable` y límites `p_nom_max` según el diseño del escenario.
