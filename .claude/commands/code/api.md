# API Design Command

Generate or review REST API endpoints for a given resource or module.

## Usage

```
/code/api [resource] [options]
```

## Arguments

- `resource`: The resource or module to generate API endpoints for
- `--framework`: Target framework (fastapi, flask, django) [default: fastapi]
- `--style`: REST style (crud, custom) [default: crud]
- `--auth`: Include authentication middleware [default: true]

## What This Does

1. Analyzes the resource model/schema
2. Generates standard CRUD endpoints
3. Adds request validation
4. Includes error handling
5. Adds OpenAPI docstrings

## Example Output

For `/code/api Product --framework fastapi`:

```python
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.orm import Session
from typing import List

from app.database import get_db
from app.models import Product
from app.schemas import ProductCreate, ProductUpdate, ProductResponse
from app.auth import get_current_user

router = APIRouter(prefix="/products", tags=["products"])


@router.get("/", response_model=List[ProductResponse])
def list_products(
    skip: int = 0,
    limit: int = 100,
    db: Session = Depends(get_db),
    current_user=Depends(get_current_user),
):
    """Retrieve a paginated list of products."""
    products = db.query(Product).offset(skip).limit(limit).all()
    return products


@router.get("/{product_id}", response_model=ProductResponse)
def get_product(
    product_id: int,
    db: Session = Depends(get_db),
    current_user=Depends(get_current_user),
):
    """Retrieve a single product by ID."""
    product = db.query(Product).filter(Product.id == product_id).first()
    if not product:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Product {product_id} not found",
        )
    return product


@router.post("/", response_model=ProductResponse, status_code=status.HTTP_201_CREATED)
def create_product(
    payload: ProductCreate,
    db: Session = Depends(get_db),
    current_user=Depends(get_current_user),
):
    """Create a new product."""
    product = Product(**payload.dict())
    db.add(product)
    db.commit()
    db.refresh(product)
    return product


@router.put("/{product_id}", response_model=ProductResponse)
def update_product(
    product_id: int,
    payload: ProductUpdate,
    db: Session = Depends(get_db),
    current_user=Depends(get_current_user),
):
    """Update an existing product."""
    product = db.query(Product).filter(Product.id == product_id).first()
    if not product:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Product {product_id} not found",
        )
    for field, value in payload.dict(exclude_unset=True).items():
        setattr(product, field, value)
    db.commit()
    db.refresh(product)
    return product


@router.delete("/{product_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_product(
    product_id: int,
    db: Session = Depends(get_db),
    current_user=Depends(get_current_user),
):
    """Delete a product by ID."""
    product = db.query(Product).filter(Product.id == product_id).first()
    if not product:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Product {product_id} not found",
        )
    db.delete(product)
    db.commit()
```

## Notes

- Always validate input with Pydantic schemas
- Use appropriate HTTP status codes
- Include pagination for list endpoints
- Protect routes with authentication by default
- Follow RESTful naming conventions (plural nouns for resources)
