Columns to drop:
    - Sensor Zenit, Sensor Azumut und Solar Zenit und Solar Azimuth wiederholen sich für jeden Sensor. Muss nur 1x behalten werden
    - Windgeschwindigkeit berechnen sqrt(U^2 + V^2) --> V und U droppen  
    - _slant_column_number_density entsprechen den nicht korrigierten Daten. ..._column_number_density sind bereits Winkelkorrigiert und sollten dementsprechend verwendet werden.   
    - Daten für CH4 enthalten sehr viele NaNs (über 80%). Erst mal droppen   
    - Werte mit niedrigem target_count evtl. geringer gewichten, da weniger Messungen zur Bildung des Mittelwertes heran gezogen wurden.  

Imputation

- Nach Ort mit Median oder alternativ interpolation zwischen Zeitpunkten für einen fixen Ort
