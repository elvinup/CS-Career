
## Load Environment Variables from file

You can have a `.env` file in your repo pre-defined that loads all your environment variables like this:

```python
from dotenv import load_dotenv

# Load config from .env file
load_dotenv()
```

