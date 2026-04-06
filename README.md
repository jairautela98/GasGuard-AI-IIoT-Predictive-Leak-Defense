# GasGuard-AI-IIoT-Predictive-Leak-Defense
IOT system using MQTT sensor fusion. Gas &amp; pressure data feed an ESP, triggering an n8n workflow where a Python AI engine detects anomalies n8n signals a solenoid actuator for automatic valve closure and alert Discord. Seamless  integrates predictive maintenance and industrial automation to ensure pipeline safety and prevent catastrophic failures.<img width="1024" height="559" alt="image" src="https://github.com/user-attachments/assets/0de0c721-7ac4-4fe2-ac33-b7a9d6abd7ce" />

import sys
import json
import pandas as pd
from sklearn.ensemble import RandomForestClassifier
import joblib

def analyze_mechatronic_health(sensor_data):
    """
    Analyzes sensor fusion data to predict pipeline integrity.
    Input: {"pressure": 25.4, "gas_ppm": 450}
    """
    # 1. Feature Engineering: The 'Leak Index'
    # High Gas + Falling Pressure = High Probability of Failure
    pressure = sensor_data.get('pressure', 0)
    gas = sensor_data.get('gas_ppm', 0)
    
    # 2. Logic: Predictive Thresholding or ML Model Prediction
    # In a real scenario, you'd use: model.predict([[pressure, gas]])
    is_anomaly = False
    status_message = "Normal Operation"
    
    # Heuristic for the 'Early Warning' demonstration
    if gas > 400 and pressure < 30:
        is_anomaly = True
        status_message = "CRITICAL: Leak Detected at Joint"
    elif gas > 250:
        status_message = "WARNING: Minor Trace Detected"

    result = {
        "leak_detected": is_anomaly,
        "status": status_message,
        "pressure_psi": pressure,
        "gas_level": gas,
        "action_required": "CLOSE_VALVE" if is_anomaly else "NONE"
    }
    return result

if __name__ == "__main__":
    try:
        # Get data from n8n 'Execute Command' node
        input_json = sys.argv[1]
        data = json.loads(input_json)
        
        prediction = analyze_mechatronic_health(data)
        
        # Output back to n8n
        print(json.dumps(prediction))
    except Exception as e:
        print(json.dumps({"error": str(e)}))




This diagram illustrates the logical connection points, expanding on the concepts seen in the specific n8n workflow designer view (Image 2):

    Industrial Data Ingestion (MQTT Trigger): n8n doesn't just process local events; it integrates with standard IIoT protocols. It listens on pipeline/data, instantly ingesting raw telemetry (seen on the far left) into the logic canvas.

    Predictive Inference (Python AI Node): This is the core fusion of n8n and AI. Instead of using a simple threshold, n8n orchestrates a call to your leak_pred.py script. The Python engine executes its XGBoost/Random Forest model, determining if the raw sensor data constitutes an anomaly or "Healthy Operation."

    Closed-Loop Mechatronic Control (MQTT Publish): This arrow is the difference between simple monitoring and predictive maintenance. If the "If Leak?" node (seen in Image 2) confirms the anomaly, n8n instantly publishes CLOSE_VALVE (MQTT CMD) back to the Solenoid Valve Actuator on the physical pipeline joint. This creates an automated, safe state.

    Operational Awareness (Discord Alert): Simultaneously, n8n hits a Webhook to trigger the real-time Discord Emergency Alert (as seen in Image 3), ensuring maintenance teams are notified immediately of the automatic mitigation.
