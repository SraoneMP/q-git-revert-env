# q-git-revert-env

A Flask-based REST API service with automated security updates via Dependabot.

**Student Email**: 21f3000245@ds.study.iitm.ac.in

## Version

8.0.0

## Security Features

- ✅ **Dependabot** configured for automated dependency updates
- ✅ Weekly security scans for vulnerabilities
- ✅ Automatic PR generation for CVE fixes

## Quick Start

```bash
pip install -r requirements.txt
python app.py
```

## API Endpoints

- `GET /health` - Health check
- `POST /api/v1/login` - User authentication
- `GET /api/v1/users` - List users
- `POST /api/v1/register` - Register new user

## Environment Variables

Copy `.env.example` to `.env` and configure:

- `DATABASE_URL` - PostgreSQL connection string
- `JWT_SECRET` - Secret for JWT tokens
- `REDIS_URL` - Redis connection for caching

## Development

```bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
flask run --debug
```

## Testing

```bash
pytest tests/ -v
```

## License

MIT
