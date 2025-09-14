# data-cleansify

Certainly! The project `data-cleansify` is designed to automate data cleaning and validation, ensuring data quality and accuracy. Below is a complete Python program for this project with comments and error handling to guide you through each step.

This program uses popular libraries like `pandas` for data manipulation and `pydantic` for validation. Ensure you have these libraries installed (`pip install pandas pydantic`) before running the program.

```python
import pandas as pd
from pydantic import BaseModel, ValidationError, validator
import numpy as np


class DataValidationModel(BaseModel):
    """
    Pydantic model for validating individual records
    """

    name: str
    age: int
    email: str
    salary: float

    @validator('name')
    def name_must_not_be_empty(cls, v):
        if not v.strip():
            raise ValueError('Name must not be empty')
        return v

    @validator('age')
    def age_must_be_positive(cls, v):
        if v <= 0:
            raise ValueError('Age must be positive')
        return v

    @validator('email')
    def email_must_be_valid(cls, v):
        if '@' not in v or '.' not in v:
            raise ValueError('Email must be valid with "@" and "."')
        return v

    @validator('salary')
    def salary_must_be_positive(cls, v):
        if v < 0:
            raise ValueError('Salary must be non-negative')
        return v


def clean_and_validate_data(df: pd.DataFrame) -> pd.DataFrame:
    """
    Cleans and validates the input data frame.
    """
    try:
        # Remove duplicates
        df = df.drop_duplicates()
        print("Removed duplicates.")

        # Fill missing numerical values with the mean
        numerical_cols = df.select_dtypes(include=[np.number]).columns
        for col in numerical_cols:
            if df[col].isnull().any():
                mean_value = df[col].mean()
                df[col] = df[col].fillna(mean_value)
                print(f"Filled missing values in '{col}' with mean: {mean_value}")

        # Fill missing categorical values with the most frequent
        categorical_cols = df.select_dtypes(include=[object]).columns
        for col in categorical_cols:
            if df[col].isnull().any():
                most_frequent = df[col].mode()[0]
                df[col] = df[col].fillna(most_frequent)
                print(f"Filled missing values in '{col}' with mode: {most_frequent}")

        # Validate data using Pydantic model
        valid_records = []
        for index, row in df.iterrows():
            try:
                validated_record = DataValidationModel(**row.to_dict())
                valid_records.append(validated_record.dict())
                print(f"Row {index} is valid.")
            except ValidationError as e:
                print(f"Row {index} is invalid: {e}")

        # Convert valid records back to DataFrame
        cleaned_df = pd.DataFrame(valid_records)
        return cleaned_df

    except Exception as e:
        print(f"An error occurred during cleaning and validation: {e}")
        return pd.DataFrame()  # Return an empty DataFrame on error


def main():
    # Sample data to demonstrate the process
    data = {
        'name': ['Alice', 'Bob', '', 'Charlie', 'David', None],
        'age': [25, 0, 30, None, None, 22],
        'email': ['alice@example.com', 'bobexample.com', 'carol@example.com', None, 'david@com', 'eve@example.com'],
        'salary': [50000, 60000, None, 70000, -100, 90000]
    }

    # Create a DataFrame
    df = pd.DataFrame(data)

    # Clean and validate data
    cleaned_df = clean_and_validate_data(df)

    if not cleaned_df.empty:
        print("\nFinal cleaned and validated data:")
        print(cleaned_df)
    else:
        print("No valid data to display.")


if __name__ == "__main__":
    main()
```

### Key Features:
- **Data Deduplication**: Removes duplicate rows from the dataset.
- **Missing Values Handling**: Fills missing numerical values with their mean and categorical values with their mode.
- **Validation with Pydantic**: Checks each record for valid `name`, `age`, `email`, and `salary`. Error messages are provided for invalid records.
- **Robust Error Handling**: Catches and logs any errors during data processing, ensuring the script doesn't crash unexpectedly.

### Usage:
- Run the program. It processes sample data defined in the `main` function.
- The validated and cleaned data is printed at the end.

This program is designed to be simple yet illustrative. Adjust sample data and model validations as needed for actual applications.