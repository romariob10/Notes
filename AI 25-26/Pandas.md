```python
import pandas as pd

data = {
    'Name': ['John', 'Anna', 'Peter', 'Alex'],
    'Age': [28, 54, 23, 57],
    'City': ['New York', 'Paris', 'Berlin', 'London']
}

df = pd.DataFrame(data)
# print(df[df['Age'] > 28])

# Первые 5 или последние 5 строк
print(df.head())
print(df.tail())
print(df.describe())
print(df.info())


# ---- CSV Read

df = pd.read_csv('info.csv')

# print(df.head())

filtered_df = df[df['Product'] == 'Product A']
print(filtered_df)

filtered_df.to_csv('new_info.csv', index=False)

# ----

data = {
    'Name': ['John', 'Anna', 'Peter', 'Linda', 'John'],
    'Age': [28, 24, None, 32, 28],
    'City': ['New York', None, 'Berlin', 'London', 'New York']
}

df = pd.DataFrame(data)
df.fillna(0, inplace=True)

df.dropna(inplace=True)

df.drop_duplicates(inplace=True)

df['Age'] = df['Age'].astype(int)

grouped = df.groupby('Age').sum()

print(grouped)
```