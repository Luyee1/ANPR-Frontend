# Security Monitoring System

## Overview

This Python application provides a real-time security monitoring interface using webcams. It leverages computer vision and machine learning models to perform Automatic Number Plate Recognition (ANPR), car brand detection, and vehicle tracking for a secure entry/exit system. The system is designed to integrate with a Firebase backend for data storage and management.

## Features

- **Real-time Video Feed**: Monitors entry and exit gates using separate webcam sources.
- **AI-Powered Detection**:
  - Detects cars in the camera feed using YOLOv8.
  - Identifies and reads license plates using a combination of a specialized YOLO model and PaddleOCR.
  - Classifies the car's brand and model using a custom-trained ResNet50 model.
- **Firebase Integration**:
  - Logs all vehicle entries and exits.
  - Checks detected license plates against a blacklist.
  - Verifies vehicle registration status (e.g., Active, Expired).
  - Manages records for registered owners and visitors.
  - Logs critical and informational alerts.
- **Gate Control**: Sends signals to Arduino controllers to open gates for authorized vehicles.
- **User-Friendly Interface**: Built with CustomTkinter, it provides an intuitive interface for security personnel, including a dialog for registering unknown visitors.

## Prerequisites

- Python 3.8+
- Two webcams (for Entry and Exit gates).
- A Google Firebase project with Firestore enabled.
- (Optional) Arduino boards connected to serial ports for physical gate control.

## Setup Instructions

### 1. Project Structure

Ensure your project files are organized as follows. You will need to create the `ANPR_model` directory and place the model files inside it.

```
.
├── ANPR_model/
│   ├── yolov8n.pt
│   ├── LPD_model/
│   │   └── weights/
│   │       └── best.pt
│   └── best_car_brand_resnet50.pth
├── Admin/
│   ├── admin_dashboard.py
│   ├── admin_main_menu.py
│   ├── ... (other admin modules)
│   └── __pycache__/
├── Security/
│   ├── security_main_menu.py
│   ├── security_monitoring.py
│   ├── ... (other security modules)
│   └── __pycache__/
├── Reset_Password/
│   ├── email_credential.py
│   ├── reset_password_interface.py
│   ├── ... (other password reset modules)
│   └── __pycache__/
├── gate_control/
│   ├── entry_gate_control/
│   │   └── entry_gate_control.ino
│   ├── exit_gate_control/
│   │   └── exit_gate_control.ino
│   └── .idea/
├── Pic/
│   ├── Logo.png
│   ├── ... (other images and icons)
├── main.py
├── login_page.py
├── firebase_config.py
├── firebase_db.py
├── ANPR.json
├── Requirement.txt
├── Montserrat.zip
├── .env
├── .gitignore
└── README.md
```

- **Admin/**: Admin dashboard and management modules.
- **Security/**: Security dashboard, monitoring, and related modules.
- **Reset_Password/**: Password reset and email credential management.
- **gate_control/**: Arduino code for entry and exit gate control.
- **Pic/**: Images and icons for the GUI.
- **main.py**: Main entry point for the application.
- **login_page.py**: Login interface.
- **firebase_config.py / firebase_db.py**: Firebase integration.
- **ANPR.json**: Firebase credentials (Does not commit).
- **Requirement.txt**: Python dependencies.
- **Montserrat.zip**: Font files for GUI.
- **.env**: Environment variables (Does not commit commit).
- **README.md**: Project documentation.

### 2. Python Environment

It is highly recommended to use a virtual environment to manage dependencies.

```bash
# Create a virtual environment
python -m venv venv

# Activate it
# On Windows:
venv\Scripts\activate
# On macOS/Linux:
source venv/bin/activate
```

### 3. Install Dependencies

Install all required Python packages. You can create a `requirements.txt` file with the following content:

**`requirements.txt`:**

```
customtkinter==5.2.2
Pillow==10.2.0
opencv-python==4.11.0.86
pyserial==3.5
torch==2.6.0+cu126
torchvision==0.21.0+cu126
ultralytics==8.3.81
paddleocr==2.10.0
paddlepaddle==3.0.0
firebase-admin==6.7.0
python-dotenv==1.0.1
cryptography==44.0.2
bcrypt==4.3.0
numpy==1.24.4
```

Use this line of code to install all the dependencies needed:
pip install customtkinter==5.2.2 Pillow==10.2.0 opencv-python==4.11.0.86 pyserial==3.5 torch==2.6.0+cu126 torchvision==0.21.0+cu126 ultralytics==8.3.81 paddleocr==2.10.0 paddlepaddle==3.0.0 firebase-admin==6.7.0 python-dotenv==1.0.1 cryptography==44.0.2 bcrypt==4.3.0 numpy==1.24.4

> **Note**: For GPU acceleration with PaddleOCR and PyTorch, you may need to install `paddlepaddle-gpu` and a CUDA-compatible version of PyTorch. Refer to their official documentation for instructions.

You need to obtain the following pre-trained models and place them in the `ANPR_model/` directory as shown in the project structure:

- `yolov8n.pt`: A standard object detection model from Ultralytics.
- `LPD_model/weights/best.pt`: A custom-trained YOLO model for license plate detection.
- `best_car_brand_resnet50.pth`: A custom-trained ResNet50 model for car brand classification.

Dataset information:
License-Plate-Data:

> used to train the license plate detection model
> sourced from roboflow

    roboflow:
        workspace: test-vaxvp
        project: license-plate-project-adaad
        version: 1
        license: CC BY 4.0
        url: https://universe.roboflow.com/test-vaxvp/license-plate-project-adaad/dataset/1

> Contains:

    total: 23531 images
    train: 20580 images (87%)
    valid: 1973 images  ( 8%)
    test : 978 images   ( 4%)

> Preprocessing steps:

    Auto-Orient: Applied to ensure images are upright.
    Resize: All images are stretched to 640×640 pixels.
    Augmentations:
        90° Rotation: Includes Clockwise, Counter-Clockwise, and Upside Down.
        Rotation: Random rotation between -15° and +15°.
        Shear: Up to ±15° Horizontal and ±15° Vertical.
        Blur: Applied up to 0.5px.

Car-Model-Data

> used to train the car model classification model
> sourced from kaggle, roboflow and google images

    roboflow:
        url: https://universe.roboflow.com/joshua-zoddd/perodua/dataset/1/download
    Kaggle:
        url: https://www.kaggle.com/datasets/occultainsights/toyota-cars-over-20k-labeled-images
        url: https://www.kaggle.com/datasets/occultainsights/honda-cars-over-11k-labeled-images

> Contains:

    14 classes which each classes contains about 500 images
    Total images: 7039
    Training images: 5627 (79.94%)
    Validation images: 1412 (20.06%)

> Preprocessing steps:

    Resize: All images are stretched to 224x224 pixels.
    Augmentations:
        Horizontal flip, color jitter, up to 500 images/class

### 4. Font Installation (Windows)

To install Montserrat for consistent GUI display:

1. Extract the Montserrat.zip file.
2. Select all .ttf files.
3. Right-click and select Install.

### 5. Arduino IDE Library Installation

To install LiquidCrystal_I2C:

1. Open Arduino IDE.
2. Go to Sketch > Include Library > Manage Libraries.
3. Search for LiquidCrystal_I2C.
4. Install LiquidCrystal I2C by Frank de Brabander.

### 6. Environment Variables Setup

Create a `.env` file in the root directory of your project to store sensitive configuration values such as encryption keys and API credentials.

1. In your project root, create a file named `.env`.
2. Add the following content to the file (replace the example values with your actual keys):

   ```env
   ENCRYPTION_KEY=your_encryption_key_here
   # Add other environment variables as needed
   ```

> **Note:**
>
> - Never commit your `.env` file to version control (e.g., GitHub) as it contains sensitive information.

### 7. How to Generate an ENCRYPTION_KEY

You need a secure key for the ENCRYPTION_KEY variable in your `.env` file. You can generate one using Python and the cryptography library:

1. **Install the cryptography library** (if you haven’t already):

   ```bash
   pip install cryptography
   ```

2. **Run the following Python code:**

   ```python
   from cryptography.fernet import Fernet
   print(Fernet.generate_key().decode())
   ```

3. **Copy the output** and paste it as the value for ENCRYPTION_KEY in your `.env` file:
   ```env
   ENCRYPTION_KEY=your_generated_key_here
   ```

### 8. How to Obtain the ANPR.json Firebase Credentials File

To connect your application to Firebase, you need a service account key file (`ANPR.json`). Follow these steps to generate and download it:

1. Go to the [Firebase Console](https://console.firebase.google.com/) and select your project.
2. In the left sidebar, click on **Project Settings** (the gear icon).
3. Go to the **Service Accounts** tab.
4. Click **Generate new private key**.
5. A JSON file will be downloaded to your computer. Rename this file to `ANPR.json` if it has a different name.
6. Move `ANPR.json` to the root directory of your project.

> **Note:** This file contains sensitive credentials. Keep it secure and never share it publicly.

### 9. Setting Up the `config` Collection in Firebase Firestore

To enable email functionality, you must create a `config` collection in your Firestore database and add an `email_credentials` document with the required fields.

#### Steps:

1. Open the [Firebase Console](https://console.firebase.google.com/) and select your project.
2. In the left sidebar, click on **Firestore Database**.
3. Click **Start collection** and enter `config` as the Collection ID.
4. Inside the `config` collection, click **Add document**.
5. Set the Document ID to `email_credentials`.
6. Add the following fields to the document:

   - `sender_email` (string): The encrypted sender email.
   - `sender_password` (string): The encrypted sender password.

   Example:

   ```
   sender_email: "gAAAAABn9APh6htlVah146noySAYm13DSJ7qYCUc2t8Y9cUnaqO_bA2K_noCIS823Sc4..."
   sender_password: "gAAAAABn9APh30jhekh6ogDjlGR3MyfV-mn5rO-9rX09exhWwKtERMD09cruwXVLgoj3b2M6..."
   ```

> **Note:**
>
> - The values for `sender_email` and `sender_password` must be encrypted using the provided utility in your project (see the `set_email_credentials` function in `Reset_Password/email_credential.py`).
> - Do not store plain text credentials in Firestore.

### Script: Set Up Email Credentials in Firestore

To make setup easier, you can use the following script to securely store your sender email and password in Firestore:

Save this as `setup_email_credentials.py` in your project root:

```python
from Reset_Password import email_credential
from firebase_db import FirebaseDB

def main():
    # Initialize Firebase
    credentials_path = "ANPR.json"
    firebase_db_instance = FirebaseDB(credentials_path)

    # Prompt user for email and password
    sender_email = input("Enter the sender email: ").strip()
    sender_password = input("Enter the sender password: ").strip()

    # Store encrypted credentials in Firestore
    email_credential.set_email_credentials(firebase_db_instance, sender_email, sender_password)
    print("Sender email and password have been securely stored in Firestore.")

if __name__ == "__main__":
    main()
```

#### How to Use

1. Save the script above as `setup_email_credentials.py` in your project root.
2. Make sure your `.env` and `ANPR.json` are set up.
3. Run the script:
   ```bash
   python setup_email_credentials.py
   ```
4. Follow the prompts to enter your sender email and password.

This will automatically encrypt your credentials and store them in the correct format in Firestore.

## Usage of the system

1. Ensure Firebase credentials (ANPR.json or .env) are set locally (do not commit to GitHub).
2. Connect webcams for entry and exit monitoring.
3. Connect the Arduino board if using physical gate control.
4. Run the system:

```
python main.py
```
