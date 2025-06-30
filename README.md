# SQLAlchemy Mastery 🐍⚡

A comprehensive guide and practical examples for mastering SQLAlchemy - Python's most powerful SQL toolkit and Object-Relational Mapping (ORM) library.

![SQLAlchemy](https://img.shields.io/badge/SQLAlchemy-2.0+-red?style=for-the-badge&logo=python)
![Python](https://img.shields.io/badge/Python-3.8+-blue?style=for-the-badge&logo=python)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

## 🎯 What You'll Learn

This repository covers everything from SQLAlchemy basics to advanced enterprise patterns:

- **Core SQLAlchemy** - Expression language and raw SQL power
- **ORM Fundamentals** - Object-relational mapping concepts
- **Advanced Relationships** - Complex associations and joins
- **Performance Optimization** - Query tuning and lazy loading strategies
- **Database Migrations** - Schema evolution with Alembic
- **Testing Patterns** - Unit and integration testing approaches
- **Production Patterns** - Connection pooling, async operations, and scalability

## 🚀 Quick Start

### Prerequisites

```bash
Python 3.8+
pip or poetry for package management
```

### Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/sqlalchemy-mastery.git
cd sqlalchemy-mastery

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Or using poetry
poetry install && poetry shell
```

### Basic Setup

```python
from sqlalchemy import create_engine, text
from sqlalchemy.orm import sessionmaker

# Create engine
engine = create_engine("sqlite:///example.db", echo=True)

# Create session
Session = sessionmaker(bind=engine)
session = Session()

# Test connection
with engine.connect() as conn:
    result = conn.execute(text("SELECT 'Hello SQLAlchemy!' as message"))
    print(result.fetchone())
```

## 📁 Project Structure

```
sqlalchemy-mastery/
├── 01-core-concepts/           # SQLAlchemy Core fundamentals
│   ├── engines_connections.py
│   ├── metadata_tables.py
│   └── raw_sql_expressions.py
├── 02-orm-basics/              # ORM fundamentals
│   ├── models.py
│   ├── sessions.py
│   └── basic_queries.py
├── 03-relationships/           # Advanced relationships
│   ├── one_to_many.py
│   ├── many_to_many.py
│   └── polymorphic.py
├── 04-advanced-queries/        # Complex querying
│   ├── joins_subqueries.py
│   ├── aggregations.py
│   └── window_functions.py
├── 05-performance/             # Optimization techniques
│   ├── lazy_loading.py
│   ├── query_optimization.py
│   └── connection_pooling.py
├── 06-migrations/              # Alembic migrations
│   ├── alembic/
│   └── migration_examples.py
├── 07-testing/                 # Testing strategies
│   ├── unit_tests.py
│   └── integration_tests.py
├── 08-async/                   # Async SQLAlchemy
│   ├── async_engine.py
│   └── async_sessions.py
├── 09-patterns/                # Enterprise patterns
│   ├── repository_pattern.py
│   ├── unit_of_work.py
│   └── data_access_layer.py
├── examples/                   # Real-world examples
│   ├── blog_app/
│   ├── ecommerce/
│   └── analytics/
├── tests/                      # Test suite
├── requirements.txt
├── pyproject.toml
└── README.md
```

## 🔧 Core Concepts

### 1. Engine and Connections

```python
from sqlalchemy import create_engine

# SQLite (file-based)
engine = create_engine("sqlite:///app.db")

# PostgreSQL
engine = create_engine("postgresql://user:pass@localhost/dbname")

# MySQL
engine = create_engine("mysql+pymysql://user:pass@localhost/dbname")

# Connection pooling
engine = create_engine(
    "postgresql://user:pass@localhost/dbname",
    pool_size=10,
    max_overflow=20,
    pool_pre_ping=True
)
```

### 2. Declarative Models

```python
from sqlalchemy import Column, Integer, String, DateTime, ForeignKey
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship
from datetime import datetime

Base = declarative_base()

class User(Base):
    __tablename__ = 'users'
    
    id = Column(Integer, primary_key=True)
    username = Column(String(50), unique=True, nullable=False)
    email = Column(String(100), unique=True, nullable=False)
    created_at = Column(DateTime, default=datetime.utcnow)
    
    posts = relationship("Post", back_populates="author")

class Post(Base):
    __tablename__ = 'posts'
    
    id = Column(Integer, primary_key=True)
    title = Column(String(200), nullable=False)
    content = Column(String, nullable=False)
    user_id = Column(Integer, ForeignKey('users.id'))
    
    author = relationship("User", back_populates="posts")
```

### 3. Session Management

```python
from sqlalchemy.orm import sessionmaker
from contextlib import contextmanager

Session = sessionmaker(bind=engine)

@contextmanager
def get_db_session():
    session = Session()
    try:
        yield session
        session.commit()
    except Exception:
        session.rollback()
        raise
    finally:
        session.close()

# Usage
with get_db_session() as session:
    user = User(username="john_doe", email="john@example.com")
    session.add(user)
```

## 🎯 Advanced Features

### Complex Relationships

```python
# Many-to-Many with Association Object
class UserRole(Base):
    __tablename__ = 'user_roles'
    user_id = Column(Integer, ForeignKey('users.id'), primary_key=True)
    role_id = Column(Integer, ForeignKey('roles.id'), primary_key=True)
    assigned_date = Column(DateTime, default=datetime.utcnow)
    
    user = relationship("User", back_populates="roles")
    role = relationship("Role", back_populates="users")

# Self-referential Relationships
class Category(Base):
    __tablename__ = 'categories'
    id = Column(Integer, primary_key=True)
    name = Column(String(50), nullable=False)
    parent_id = Column(Integer, ForeignKey('categories.id'))
    
    children = relationship("Category", backref=backref('parent', remote_side=[id]))
```

### Query Optimization

```python
# Eager Loading
users_with_posts = session.query(User).options(
    joinedload(User.posts)
).all()

# Batch Loading
users = session.query(User).options(
    selectinload(User.posts)
).all()

# Query with Complex Joins
result = session.query(User, func.count(Post.id).label('post_count')).\\
    outerjoin(Post).\\
    group_by(User.id).\\
    having(func.count(Post.id) > 5).\\
    all()
```

## 🧪 Testing

Run the test suite:

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=src --cov-report=html

# Run specific test file
pytest tests/test_models.py

# Run with database setup
pytest --db-url=sqlite:///test.db
```

## 📊 Performance Tips

1. **Use Connection Pooling**: Configure appropriate pool sizes for your application
2. **Optimize Queries**: Use `joinedload()` and `selectinload()` to avoid N+1 problems
3. **Batch Operations**: Use `bulk_insert_mappings()` for large datasets
4. **Index Strategy**: Create appropriate database indexes for frequent queries
5. **Lazy Loading**: Configure relationship loading strategies based on usage patterns

## 🔄 Migration Management

```bash
# Initialize Alembic
alembic init alembic

# Create migration
alembic revision --autogenerate -m "Add user table"

# Apply migrations
alembic upgrade head

# Rollback migration
alembic downgrade -1
```

## 🌟 Real-World Examples

### Blog Application
- User authentication and authorization
- Post creation and management
- Comment system with threading
- Tag-based categorization

### E-commerce Platform
- Product catalog with variants
- Shopping cart and order management
- Payment processing integration
- Inventory tracking

### Analytics Dashboard
- Time-series data processing
- Aggregation and reporting
- Data visualization preparation
- Performance monitoring

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Development Setup

```bash
# Install development dependencies
pip install -e ".[dev]"

# Run pre-commit hooks
pre-commit install

# Format code
black src/ tests/
isort src/ tests/

# Run linting
flake8 src/ tests/
mypy src/
```

## 📚 Resources

- [SQLAlchemy Official Documentation](https://docs.sqlalchemy.org/)
- [SQLAlchemy 2.0 Migration Guide](https://docs.sqlalchemy.org/en/20/changelog/migration_20.html)
- [Alembic Documentation](https://alembic.sqlalchemy.org/)
- [Fast API with SQLAlchemy](https://fastapi.tiangolo.com/tutorial/sql-databases/)

## 📈 Learning Path

1. **Beginner**: Start with Core concepts and basic ORM
2. **Intermediate**: Master relationships and query optimization
3. **Advanced**: Implement async patterns and enterprise architectures
4. **Expert**: Contribute to open source and build scalable applications

## ⚡ Quick Commands

```bash
# Start development database
docker-compose up -d postgres

# Run examples
python examples/blog_app/main.py

# Generate sample data
python scripts/seed_data.py

# Run performance benchmarks
python benchmarks/query_performance.py
```

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- SQLAlchemy team for creating an amazing ORM
- Python community for continuous support
- Contributors who help improve this resource

---

**Happy Learning! 🎉**

If you find this repository helpful, please consider giving it a ⭐️ and sharing it with others!
