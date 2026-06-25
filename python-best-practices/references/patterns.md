# Python patterns

Before/after for the smells that are hardest to catch by eye. Each pair shows the smell, then the typed or composed version.

## Dynamic attribute access for known fields

```python
# Bad: getattr defeats the checker and hides typos. "amont" fails silently.
value = getattr(record, field_name, None)
if getattr(record, "is_seeded", False):
    ...

# Good: typed access on a modelled shape.
@dataclass(frozen=True)
class Record:
    amount: Decimal
    is_seeded: bool

if record.is_seeded:
    ...
```

Reach for `getattr` only when the attribute name is genuinely computed at runtime. A name you typed as a string literal is known at design time, so access it directly.

## Raw dict as a record

```python
# Bad: shape is implicit, every reader guesses the keys and types.
def summarize(doc: dict) -> dict:
    return {"pages": len(doc["blocks"]), "tenant": doc["tenant_id"]}

# Good: the shape is the type.
@dataclass(frozen=True)
class Document:
    blocks: list[Block]
    tenant_id: TenantId

@dataclass(frozen=True)
class Summary:
    pages: int
    tenant: TenantId

def summarize(doc: Document) -> Summary:
    return Summary(pages=len(doc.blocks), tenant=doc.tenant_id)
```

## Magic string for a closed set

```python
# Bad: any typo is a valid string, no checker help.
if doc.status == "completed":
    ...

# Good: a string enum, referenced everywhere the set is used.
class DocStatus(str, Enum):
    PENDING = "pending"
    COMPLETED = "completed"
    FAILED = "failed"

if doc.status is DocStatus.COMPLETED:
    ...
```

`Literal["completed", "failed"]` is fine for a one-off local parameter. Once the set is named, shared, or carries behaviour, make it an enum.

## All-Optional dataclass hides illegal states

```python
# Bad: which combination is valid? Nothing stops result=None with error set, or both set.
@dataclass
class FetchResult:
    value: bytes | None = None
    error: str | None = None
    retrying: bool = False

# Good: model the variants so impossible states cannot be built.
@dataclass(frozen=True)
class Fetched:
    value: bytes

@dataclass(frozen=True)
class FetchFailed:
    error: str

FetchResult = Fetched | FetchFailed

match result:
    case Fetched(value):
        ...
    case FetchFailed(error):
        ...
```

## isinstance ladder instead of dispatch

```python
# Bad: every new shape edits this ladder, and the checker can't prove it is exhaustive.
def area(shape) -> float:
    if isinstance(shape, Circle):
        return math.pi * shape.r ** 2
    elif isinstance(shape, Square):
        return shape.side ** 2
    raise ValueError("unknown shape")

# Good: polymorphism via a Protocol; each shape owns its rule.
class Shape(Protocol):
    def area(self) -> float: ...

@dataclass(frozen=True)
class Circle:
    r: float
    def area(self) -> float:
        return math.pi * self.r ** 2

@dataclass(frozen=True)
class Square:
    side: float
    def area(self) -> float:
        return self.side ** 2
```

## Composition over inheritance

```python
# Bad: inheritance to reuse behaviour creates a rigid hierarchy and a fragile base.
class JsonReportingService(BaseService):
    def run(self):
        data = super().fetch()
        return self.to_json(data)

# Good: compose collaborators, depend on a Protocol, inject them.
class Fetcher(Protocol):
    def fetch(self) -> Data: ...

class Formatter(Protocol):
    def format(self, data: Data) -> str: ...

@dataclass(frozen=True)
class ReportingService:
    fetcher: Fetcher
    formatter: Formatter

    def run(self) -> str:
        return self.formatter.format(self.fetcher.fetch())
```

Use inheritance only for a genuine is-a with shared behaviour. When you just need an interface, a Protocol is enough.

## Mutable default argument

```python
# Bad: the list is created once and shared across every call.
def add_tag(tag: str, tags: list[str] = []) -> list[str]:
    tags.append(tag)
    return tags

# Good: default to None, build inside.
def add_tag(tag: str, tags: list[str] | None = None) -> list[str]:
    tags = list(tags or [])
    tags.append(tag)
    return tags
```

## Validate at the boundary, don't cast

```python
# Bad: cast asserts a shape the checker never verified; bad input crashes deep inside.
def handle(payload: dict) -> None:
    event = cast(IngestionEvent, payload)
    process(event.document_id)

# Good: parse at the boundary, then trust the type inside.
def handle(payload: dict) -> None:
    event = IngestionEvent.model_validate(payload)  # raises on bad input, here
    process(event.document_id)
```

## In-body comment block hides a helper

```python
# Bad: each comment narrates the next chunk. The comments are the function names trying to escape.
def build_summary(doc: Document) -> Summary:
    # Drop blocks that failed parsing first, otherwise they skew the
    # per-page counts and the dense-page calculation below.
    valid_blocks = [b for b in doc.blocks if b.status is BlockStatus.PARSED]

    # Group the remaining blocks by page so we can report per-page totals.
    by_page: dict[int, list[Block]] = {}
    for block in valid_blocks:
        by_page.setdefault(block.page, []).append(block)

    # A page is "dense" when it carries more than the threshold of blocks.
    dense = [page for page, blocks in by_page.items() if len(blocks) > DENSE_PAGE_THRESHOLD]

    return Summary(pages=len(by_page), dense_pages=dense)

# Good: the comment for each step becomes a named helper. The top function reads as its steps,
# and each helper is independently testable.
def build_summary(doc: Document) -> Summary:
    by_page = group_by_page(parsed_blocks(doc))
    return Summary(pages=len(by_page), dense_pages=dense_pages(by_page))

def parsed_blocks(doc: Document) -> list[Block]:
    return [b for b in doc.blocks if b.status is BlockStatus.PARSED]

def group_by_page(blocks: list[Block]) -> dict[int, list[Block]]:
    by_page: dict[int, list[Block]] = {}
    for block in blocks:
        by_page.setdefault(block.page, []).append(block)
    return by_page

def dense_pages(by_page: dict[int, list[Block]]) -> list[int]:
    return [page for page, blocks in by_page.items() if len(blocks) > DENSE_PAGE_THRESHOLD]
```

The comment that explained each step is replaced by the helper's name. A docstring on `build_summary` is still fine; the narration inside the body is what moves into names.
