## Let's build something

To give you a better idea what's possible with React Server, let's build a rudimentary CMS, step by step.

### The Navigation

Let's start by building a headless navigation that we can consume. 
We won't bother adding authentication at the beginning. Let's make everyone an admin for now. It will be trivial to add authentication later.

We dont' need much logic to implement a serverside navigation. We basically need CRUD functionality to add, remove, edit or delete entries to the navigation.

We'll keep things simple and basic. Let's start by scaffolding a serverside component.

```tsx
import { useState } from '@state-less/react-server';
import { ServerSideProps } from './ServerSideProps';
import { v4 } from 'uuid';

type NavigationEntry = {
    id: string;
    path: string;
    title: string;
};

export const isEntry = (entry: any): entry is NavigationEntry => {
    return entry.id && entry.path && entry.title;
};

export const Navigation = () => {
    const [entries, setEntries] = useState<NavigationEntry[]>([], {
        key: 'entries',
    });

    const addEntry = (entry) => {
        const id = v4();
        const newEntry = { ...entry, id };
        if (!isEntry(newEntry)) {
            throw new Error('Invalid entry');
        }
        if (entries.find((e) => e.path === entry.path)) {
            throw new Error('Entry already exists');
        }
        setEntries([...entries, newEntry]);
    };

    const removeEntry = (id) => {
        setEntries(entries.filter((entry) => entry.id !== id));
    };

    return (
        <ServerSideProps
            entries={entries}
            addEntry={addEntry}
            removeEntry={removeEntry}
        />
    );
};

```

We'll add an option to edit an entry later on. Let's just get the basic functionality of adding and removing entries to a list done.

Head over to your client side code and create a `ServerNavigation.tsx` file under `src/server-components/`.

The frontend code looks a bit more verbose, but don't get intimidated by it. It's actually very simple.

*src/server-components/*
```tsx
import {
  IconButton,
  List,
  ListItem,
  ListItemSecondaryAction,
  ListItemText,
  TextField,
  Alert,
} from '@mui/material';
import { useComponent } from '@state-less/react-client/dist/index';
import { useState } from 'react';
import { localClient } from '../lib/client';

import CloseIcon from '@mui/icons-material/Close';

/** This should check if the path contains a / and also that it doesn't contain any special characters */
const isValidPath = (path: string) => {
  return /^[0-9A-Za-z_-]+$/.test(path);
};

const errors = (messages) => {
  return messages
    .filter(([, isErr]) => isErr)
    .map(([msg]) => msg)
    .join(', ');
};
export const ServerNavigation = () => {
  const [component, { error, loading }] = useComponent('navigation', {
    client: localClient,
  });
  const [title, setTitle] = useState('');
  const [path, setPath] = useState('');

  const errs = [
    ['Invalid path', !isValidPath(path)],
    ['Title and path are required', !title || !path],
  ];

  const titleMessage = errors(errs.slice(1));
  const pathMessage = errors(errs);
  
  return (
    <div>
      {error && <Alert severity="error">{error.message}</Alert>}
      <TextField
        label="Title"
        value={title}
        onChange={(e) => setTitle(e.target.value)}
        error={!!titleMessage}
        helperText={titleMessage}
      ></TextField>
      <TextField
        label="Path"
        value={path}
        onChange={(e) => setPath(e.target.value)}
        error={!!pathMessage}
        helperText={pathMessage}
      ></TextField>
      <List>
        {component?.props?.entries.map((entry) => {
          return (
            <ListItem>
              <ListItemText primary={entry.title} secondary={entry.path} />
              <ListItemSecondaryAction>
                <IconButton
                  onClick={() => component.props.removeEntry(entry.id)}
                >
                  <CloseIcon />
                </IconButton>
              </ListItemSecondaryAction>
            </ListItem>
          );
        })}
      </List>
      <Button onClick={() => component.props.addEntry({ title, path })}>
        Add Entry
      </Button>
    </div>
  );
};
```