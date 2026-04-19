# Scaffold Command

Generate boilerplate code for common patterns and project structures.

## Usage

```
/scaffold <type> [name] [options]
```

## Arguments

- `type`: The type of scaffold to generate (api, model, service, component, test, cli)
- `name`: Name for the generated code
- `options`: Additional configuration flags

## Examples

```
/scaffold api UserController
/scaffold model Product --fields "name:str,price:float,stock:int"
/scaffold service EmailService
/scaffold test UserService
```

## What This Does

1. Detects the project type and framework (FastAPI, Flask, Django, plain Python)
2. Generates idiomatic boilerplate matching existing code style
3. Creates associated test file if `--with-tests` flag is provided
4. Follows naming conventions found in the codebase

## Scaffold Types

### `api` — REST endpoint handler

```python
# Generated: routers/user_controller.py
from fastapi import APIRouter, HTTPException, status
from typing import List
from ..schemas.user import UserCreate, UserRead, UserUpdate
from ..services.user_service import UserService

router = APIRouter(prefix="/users", tags=["users"])

@router.get("/", response_model=List[UserRead])
async def list_users():
    return await UserService.get_all()

@router.get("/{user_id}", response_model=UserRead)
async def get_user(user_id: int):
    user = await UserService.get_by_id(user_id)
    if not user:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="User not found")
    return user

@router.post("/", response_model=UserRead, status_code=status.HTTP_201_CREATED)
async def create_user(payload: UserCreate):
    return await UserService.create(payload)

@router.put("/{user_id}", response_model=UserRead)
async def update_user(user_id: int, payload: UserUpdate):
    user = await UserService.update(user_id, payload)
    if not user:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="User not found")
    return user

@router.delete("/{user_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_user(user_id: int):
    deleted = await UserService.delete(user_id)
    if not deleted:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="User not found")
```

### `model` — Data model / ORM class

```python
# Generated: models/product.py
from sqlalchemy import Column, Integer, String, Float, DateTime, func
from ..database import Base

class Product(Base):
    __tablename__ = "products"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(255), nullable=False)
    price = Column(Float, nullable=False)
    stock = Column(Integer, default=0)
    created_at = Column(DateTime, server_default=func.now())
    updated_at = Column(DateTime, server_default=func.now(), onupdate=func.now())

    def __repr__(self) -> str:
        return f"<Product id={self.id} name={self.name!r}>"
```

### `service` — Business logic service layer

```python
# Generated: services/email_service.py
import smtplib
import logging
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from typing import Optional
from ..config import settings

logger = logging.getLogger(__name__)

class EmailService:
    @staticmethod
    async def send(
        to: str,
        subject: str,
        body: str,
        html: Optional[str] = None,
    ) -> bool:
        """Send an email. Returns True on success, False on failure."""
        msg = MIMEMultipart("alternative")
        msg["Subject"] = subject
        msg["From"] = settings.SMTP_FROM
        msg["To"] = to
        msg.attach(MIMEText(body, "plain"))
        if html:
            msg.attach(MIMEText(html, "html"))
        try:
            with smtplib.SMTP(settings.SMTP_HOST, settings.SMTP_PORT) as server:
                server.starttls()
                server.login(settings.SMTP_USER, settings.SMTP_PASSWORD)
                server.sendmail(settings.SMTP_FROM, to, msg.as_string())
            return True
        except Exception as exc:
            logger.error("Failed to send email to %s: %s", to, exc)
            return False
```

## Notes

- Scaffold output is a **starting point** — review and adjust to your domain
- Run `/lint` after scaffolding to catch style issues
- Run `/test` to generate matching test stubs
