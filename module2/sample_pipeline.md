

# ✅ **Final Correct Version**

```python
import os
import sys
import pandas as pd
import numpy as np
import secrets
from typing import List, Tuple

from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.compose import ColumnTransformer
from sklearn.impute import SimpleImputer
from sklearn.pipeline import Pipeline

import hashlib
from cryptography.fernet import Fernet
import pickle


# -------------------------
# Configuration
# -------------------------
MODEL_DIR = os.environ.get("MODEL_DIR", "/secure_storage/models")
MODEL_PATH = os.path.join(MODEL_DIR, "finalized_model.enc")
HASH_PATH = os.path.join(MODEL_DIR, "model_hash.txt")

EXPECTED_COLUMNS: List[str] = ["feature1", "feature2", "feature3_cat", "target"]
TARGET_COLUMN = "target"

os.makedirs(MODEL_DIR, exist_ok=True)


# -------------------------
# Utility Helpers
# -------------------------
def get_cipher_from_env() -> Fernet:
    key = os.environ.get("MODEL_KEY")
    if not key:
        raise ValueError("MODEL_KEY missing. Please export a valid Fernet key.")

    try:
        return Fernet(key.encode())
    except Exception as e:
        raise ValueError("Invalid MODEL_KEY provided.") from e


def sha256_hex(data: bytes) -> str:
    return hashlib.sha256(data).hexdigest()


# -------------------------
# Data Loading & Validation
# -------------------------
def load_dataset(path: str) -> pd.DataFrame:
    try:
        return pd.read_csv(path)
    except Exception:
        raise ValueError("Dataset corrupted or unreadable.")


def validate_schema(df: pd.DataFrame, expected: List[str], target: str) -> None:
    missing = [c for c in expected if c not in df.columns]
    if missing:
        raise ValueError(f"Missing required columns: {missing}")
    if target not in df.columns:
        raise ValueError(f"Target column '{target}' not found.")


# -------------------------
# Cleaning & Preprocessing
# -------------------------
def remove_duplicates(df: pd.DataFrame) -> pd.DataFrame:
    return df.drop_duplicates()


def detect_and_filter_outliers(df: pd.DataFrame, numeric_cols: List[str], threshold: float = 3.0) -> pd.DataFrame:
    if not numeric_cols:
        return df
    z = (df[numeric_cols] - df[numeric_cols].mean()) / df[numeric_cols].std(ddof=0)
    mask = (np.abs(z) < threshold).all(axis=1)
    return df[mask]


def split_features_target(df: pd.DataFrame, target: str):
    X = df.drop(columns=[target])
    y = df[target]
    return X, y


def build_pipeline(X: pd.DataFrame) -> Pipeline:
    numeric_cols = X.select_dtypes(include=[np.number]).columns.tolist()
    categorical_cols = X.select_dtypes(exclude=[np.number]).columns.tolist()

    numeric_transformer = Pipeline(steps=[
        ("imputer", SimpleImputer(strategy="median")),
        ("scaler", StandardScaler())
    ])

    categorical_transformer = Pipeline(steps=[
        ("imputer", SimpleImputer(strategy="most_frequent")),
        ("encoder", OneHotEncoder(handle_unknown="ignore"))
    ])

    preprocessor = ColumnTransformer(
        transformers=[
            ("num", numeric_transformer, numeric_cols),
            ("cat", categorical_transformer, categorical_cols),
        ]
    )

    model = LogisticRegression(max_iter=500)

    return Pipeline(steps=[
        ("preprocessor", preprocessor),
        ("model", model)
    ])


# -------------------------
# Secure Save / Load
# -------------------------
def secure_save_model(model: Pipeline, model_path: str, hash_path: str, cipher: Fernet) -> None:
    # Serialize with pickle
    model_bytes = pickle.dumps(model)

    # Encrypt
    encrypted = cipher.encrypt(model_bytes)

    # Save encrypted model
    with open(model_path, "wb") as f:
        f.write(encrypted)

    # Save integrity hash (of encrypted bytes)
    with open(hash_path, "w") as f:
        f.write(sha256_hex(encrypted))


def secure_load_model(model_path: str, hash_path: str, cipher: Fernet) -> Pipeline:
    # Read encrypted model
    with open(model_path, "rb") as f:
        encrypted = f.read()

    # Integrity check
    current_hash = sha256_hex(encrypted)
    with open(hash_path, "r") as f:
        stored_hash = f.read().strip()

    if current_hash != stored_hash:
        raise ValueError("Integrity check FAILED: model may have been tampered with.")

    # Decrypt + load
    decrypted = cipher.decrypt(encrypted)
    return pickle.loads(decrypted)


# -------------------------
# Main Pipeline Flow
# -------------------------
def main():
    # 1) Load data
    df = load_dataset("user_data.csv")

    # 2) Schema validation
    validate_schema(df, EXPECTED_COLUMNS, TARGET_COLUMN)

    # 3) Cleaning
    df = remove_duplicates(df)

    numeric_cols = df.select_dtypes(include=[np.number]).columns.tolist()
    df = detect_and_filter_outliers(df, numeric_cols, threshold=3.0)

    # 4) Split X/y
    X, y = split_features_target(df, TARGET_COLUMN)

    # 5) Secure random state using secrets.token_bytes
    # Produce a random 32-bit integer from 4 secure random bytes
    random_state = int.from_bytes(secrets.token_bytes(4), "big")

    # 6) Train/test split
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=random_state
    )

    # 7) Build + train pipeline
    pipeline = build_pipeline(X_train)
    pipeline.fit(X_train, y_train)

    # 8) Encrypt + save model
    cipher = get_cipher_from_env()
    secure_save_model(pipeline, MODEL_PATH, HASH_PATH, cipher)

    # 9) Load securely
    loaded_model = secure_load_model(MODEL_PATH, HASH_PATH, cipher)

    # 10) Evaluate
    accuracy = loaded_model.score(X_test, y_test)
    print(f"Model Accuracy: {accuracy:.2f}")
    print("Model saved to:", MODEL_PATH)


if __name__ == "__main__":
    if os.environ.get("MODEL_KEY") is None:
        print("ERROR: MODEL_KEY not set.", file=sys.stderr)
        print("Generate one using:", file=sys.stderr)
        print('python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"')
        sys.exit(1)

    main()
```

## ✅ 2. Diagram of the pipeline flow

### High-level pipeline flow

```text
                ┌───────────────────────────┐
                │        user_data.csv      │
                └────────────┬──────────────┘
                             │
                  Load + basic validation
                             │
                             ▼
                ┌───────────────────────────┐
                │      load_dataset()       │
                └────────────┬──────────────┘
                             │
                             ▼
                ┌───────────────────────────┐
                │    validate_schema()      │
                └────────────┬──────────────┘
                             │
                      Cleaning steps
                             │
       ┌─────────────────────┴─────────────────────┐
       ▼                                           ▼
┌───────────────┐                         ┌─────────────────────┐
│ remove_        │                         │ detect_and_filter_ │
│ duplicates()   │                         │   _outliers()      │
└───────┬────────┘                         └──────────┬─────────┘
        │                                            │
        └────────────────────────────────────────────┘
                             │
                             ▼
                ┌───────────────────────────┐
                │  split_features_target()  │
                │         (X, y)            │
                └────────────┬──────────────┘
                             │
                             ▼
                 Secure random state
        (secrets.token_bytes(4) → int.from_bytes)
                             │
                             ▼
                ┌───────────────────────────┐
                │   train_test_split()      │
                │   (X_train, X_test, ...)  │
                └────────────┬──────────────┘
                             │
                             ▼
                ┌───────────────────────────┐
                │     build_pipeline()      │
                │  (preprocess + model)     │
                └────────────┬──────────────┘
                             │
                             ▼
                ┌───────────────────────────┐
                │   pipeline.fit()          │
                │   (train model)           │
                └────────────┬──────────────┘
                             │
                             ▼
           ┌─────────────────────────────────────┐
           │     secure_save_model()             │
           │  - pickle.dumps()                   │
           │  - Fernet.encrypt()                 │
           │  - sha256_hex(encrypted) → hash     │
           └─────────────────┬───────────────────┘
                             │
                     Saved to disk:
              MODEL_PATH (.enc)  +  HASH_PATH
                             │
                             ▼
           ┌─────────────────────────────────────┐
           │     secure_load_model()             │
           │  - read encrypted bytes             │
           │  - verify SHA-256 hash              │
           │  - Fernet.decrypt()                 │
           │  - pickle.loads()                   │
           └─────────────────┬───────────────────┘
                             │
                             ▼
                ┌───────────────────────────┐
                │   loaded_model.score()    │
                │     (X_test, y_test)      │
                └───────────────────────────┘
```
