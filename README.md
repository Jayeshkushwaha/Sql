## Sql in React Native:

#### Step 1: Setting up the React Native Project
First, create a new React Native project:

```bash
npx react-native init SQLiteDemo
cd SQLiteDemo
```

#### Step 2: Installing Dependencies
Install the necessary packages for SQLite:

```bash
npm install react-native-sqlite-storage
```

Link the SQLite storage package (for React Native versions below 0.60, use `react-native link` instead):

```bash
npx pod-install
```

#### Step 3: Setting up SQLite
Create a `database.js` file in the root of your project to handle SQLite operations:

```javascript
// database.js
import SQLite from 'react-native-sqlite-storage';

const db = SQLite.openDatabase(
  {
    name: 'MainDB',
    location: 'default',
  },
  () => {},
  error => {
    console.log("ERROR: " + error);
  }
);

export const createTable = () => {
  db.transaction(txn => {
    txn.executeSql(
      `CREATE TABLE IF NOT EXISTS Items (id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT)`,
      [],
      (sqlTxn, res) => {
        console.log("Table created successfully");
      },
      error => {
        console.log("Error on creating table " + error.message);
      }
    );
  });
};

export const getItems = (setItems) => {
  db.transaction(txn => {
    txn.executeSql(
      `SELECT * FROM Items`,
      [],
      (sqlTxn, res) => {
        console.log("Items retrieved successfully");
        let len = res.rows.length;

        if (len > 0) {
          let results = [];
          for (let i = 0; i < len; i++) {
            let item = res.rows.item(i);
            results.push({ id: item.id, name: item.name });
          }
          setItems(results);
        } else {
          setItems([]);
        }
      },
      error => {
        console.log("Error on retrieving items " + error.message);
      }
    );
  });
};

export const addItem = (name, getItems) => {
  db.transaction(txn => {
    txn.executeSql(
      `INSERT INTO Items (name) VALUES (?)`,
      [name],
      (sqlTxn, res) => {
        console.log(`${name} item added successfully`);
        getItems();
      },
      error => {
        console.log("Error on adding item " + error.message);
      }
    );
  });
};

export const deleteItem = (id, getItems) => {
  db.transaction(txn => {
    txn.executeSql(
      `DELETE FROM Items WHERE id = ?`,
      [id],
      (sqlTxn, res) => {
        console.log(`Item with id ${id} deleted successfully`);
        getItems();
      },
      error => {
        console.log("Error on deleting item " + error.message);
      }
    );
  });
};

export const updateItem = (id, newName, getItems) => {
  db.transaction(txn => {
    txn.executeSql(
      `UPDATE Items SET name = ? WHERE id = ?`,
      [newName, id],
      (sqlTxn, res) => {
        console.log(`Item with id ${id} updated successfully`);
        getItems();
      },
      error => {
        console.log("Error on updating item " + error.message);
      }
    );
  });
};
```

#### Step 4: Creating the Main App Component
Edit `App.js` to include basic CRUD operations:

```javascript
// App.js
import React, { useEffect, useState } from 'react';
import { SafeAreaView, View, Text, TextInput, Button, FlatList, StyleSheet } from 'react-native';
import { createTable, getItems, addItem, deleteItem, updateItem } from './database';

const App = () => {
  const [name, setName] = useState('');
  const [items, setItems] = useState([]);
  const [editingId, setEditingId] = useState(null);

  useEffect(() => {
    createTable();
    getItems(setItems);
  }, []);

  const handleAddOrUpdateItem = () => {
    if (name.length > 0) {
      if (editingId === null) {
        addItem(name, () => getItems(setItems));
      } else {
        updateItem(editingId, name, () => getItems(setItems));
        setEditingId(null);
      }
      setName('');
    }
  };

  const handleEditItem = (item) => {
    setName(item.name);
    setEditingId(item.id);
  };

  const handleDeleteItem = (id) => {
    deleteItem(id, () => {
      getItems(setItems);
      setName('');
      setEditingId(null);
    });
  };

  const renderItem = ({ item }) => (
    <View style={styles.item}>
      <Text>{item.name}</Text>
      <View style={styles.buttons}>
        <Button title="Edit" onPress={() => handleEditItem(item)} />
        <Button title="Delete" onPress={() => handleDeleteItem(item.id)} />
      </View>
    </View>
  );

  return (
    <SafeAreaView style={styles.container}>
      <TextInput
        placeholder="Enter item name"
        value={name}
        onChangeText={setName}
        style={styles.input}
      />
      <Button
        title={editingId === null ? "Add Item" : "Update Item"}
        onPress={handleAddOrUpdateItem}
      />
      <FlatList
        data={items}
        renderItem={renderItem}
        keyExtractor={item => item.id.toString()}
      />
    </SafeAreaView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
  },
  input: {
    borderWidth: 1,
    borderColor: '#000',
    padding: 10,
    marginBottom: 10,
  },
  item: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    padding: 10,
    borderBottomWidth: 1,
    borderBottomColor: '#ccc',
  },
  buttons: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    width: 100,
  },
});

export default App;
```

#### Step 5: Running the App
Finally, run your React Native app:

```bash
npx react-native run-android
# or
npx react-native run-ios
```

This will start the application, where you can add, display, and delete items using SQLite for local storage. 

### Summary
You now have a basic React Native app using SQLite for local data storage. This setup includes initializing the SQLite database, creating a table, and performing basic CRUD operations.
