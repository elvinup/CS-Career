
## Install

Container that puts data into /tmp/arangodb

```bash
podman run -d -p 8529:8529 -e ARANGO_ROOT_PASSWORD=openSesame \
    -v /tmp/arangodb:/var/lib/arangodb3 --name arangodb \
    arangodb/arangodb:3.11.4
```

## Basic Usage

`RETURN NEW` added to the end of a query shows the value handled in JSON

### Retrieve Document

```
RETURN DOCUMENT("users/9883")
```

### Insert Document

```
INSERT { _key: "10074", name: "James Hendrix", age: 69 } INTO users 
```

### Iterate and Sort Documents

```
FOR user IN users SORT user.age DESC RETURN user
```

### Modify Documents

Does a partial update on an existing record. Replacing whole doc instead uses `REPLACE`
```
UPDATE "9915" WITH { age: 40 } IN users
```

### Remove Documents

```
REMOVE "9883" IN users
```
