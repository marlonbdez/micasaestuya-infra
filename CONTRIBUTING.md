# Contributing to Mi Casa Es Tuya Infrastructure

Thank you for your interest in contributing! 🎉

## Getting Started

1. **Fork** the repository
2. **Clone** your fork locally
3. **Create a branch** for your feature or fix:
   ```bash
   git checkout -b feat/my-awesome-improvement
   ```
4. **Make your changes** and test them
5. **Commit** with clear messages
6. **Push** to your fork
7. **Open a Pull Request** describing your changes

## What's Here?

This repo contains:
- **Docker Compose** configuration for the full stack
- **MongoDB** initialization and seed data
- **Region data** for different countries
- **Development environment** setup

## Development Setup

### Prerequisites
- Docker & Docker Compose
- Mac/Linux (Windows with WSL2 recommended)

### Quick Start

```bash
# Start the full stack
docker-compose up -d

# View logs
docker-compose logs -f

# Stop everything
docker-compose down
```

### Services Running
- **API**: http://localhost:3001 (Node.js + Express)
- **Frontend**: http://localhost:3000 (Nuxt)
- **MongoDB**: mongodb://localhost:27017
- **Redis**: redis://localhost:6379

### Accessing Services

**MongoDB**
```bash
# From inside Docker container
docker-compose exec mongo mongosh
```

**Redis**
```bash
# From inside Docker container
docker-compose exec redis redis-cli
```

## File Structure

```
infra/
├── docker-compose.yml     # Main orchestration
├── .env.example          # Environment template
├── seed/
│   ├── mongo-init.js     # Initialization script
│   ├── regions.json      # Region data
│   ├── regions_cu.json   # Cuba regions
│   └── regions_do.json   # Dominican Republic regions
└── Dockerfile            # Custom images (if needed)
```

## Adding New Regions

1. Create a new JSON file: `seed/regions_XX.json`
2. Follow the format:
   ```json
   [
     {
       "code": "01",
       "name": "Region Name",
       "country": "XX"
     }
   ]
   ```
3. Update `mongo-init.js` to include your file
4. Test with: `docker-compose down && docker-compose up -d`

## Environment Variables

Copy `.env.example` to `.env`:
```bash
cp .env.example .env
```

Update values for your environment.

⚠️ **Never commit `.env` to git** - it contains secrets.

## Commit Messages

Use clear, descriptive commit messages:
```
feat: Add support for new regions
fix: Resolve MongoDB connection timeout
docs: Update infrastructure documentation
test: Add validation for seed data
refactor: Simplify docker-compose configuration
```

## Pull Request Guidelines

- **Title**: Keep it short and descriptive
- **Description**: Explain what and why
- **Testing**: Test with `docker-compose up`
- **Documentation**: Update docs if adding features
- **One feature per PR**: Keep PRs focused

## Troubleshooting

### Services won't start
```bash
docker-compose down -v  # Remove volumes
docker-compose up -d    # Start fresh
```

### Port already in use
Edit `docker-compose.yml` to use different ports

### MongoDB connection fails
```bash
docker-compose logs mongo  # Check logs
docker-compose restart mongo
```

## Questions?

Open an **Issue** if you have questions. We're here to help!

## License

By contributing, you agree that your contributions will be licensed under the AGPL-3.0 License.

---

**Happy coding!** 🚀
