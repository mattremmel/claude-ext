# Common Patterns - JavaScript/TypeScript

## API Response Interface

```typescript
interface ApiResponse<T> {
  success: boolean
  data?: T
  error?: string
  meta?: {
    total: number
    page: number
    limit: number
  }
}
```

Usage:

```typescript
async function fetchUsers(): Promise<ApiResponse<User[]>> {
  try {
    const users = await db.users.findAll()
    return {
      success: true,
      data: users,
      meta: { total: users.length, page: 1, limit: 50 }
    }
  } catch (error) {
    return {
      success: false,
      error: 'Failed to fetch users'
    }
  }
}
```

## React Hooks - useDebounce

```typescript
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value)

  useEffect(() => {
    const handler = setTimeout(() => setDebouncedValue(value), delay)
    return () => clearTimeout(handler)
  }, [value, delay])

  return debouncedValue
}
```

Usage:

```typescript
function SearchComponent() {
  const [searchTerm, setSearchTerm] = useState('')
  const debouncedSearch = useDebounce(searchTerm, 300)

  useEffect(() => {
    if (debouncedSearch) {
      performSearch(debouncedSearch)
    }
  }, [debouncedSearch])

  return <input value={searchTerm} onChange={(e) => setSearchTerm(e.target.value)} />
}
```

## Repository Pattern

```typescript
interface Repository<T> {
  findAll(filters?: Filters): Promise<T[]>
  findById(id: string): Promise<T | null>
  create(data: CreateDto): Promise<T>
  update(id: string, data: UpdateDto): Promise<T>
  delete(id: string): Promise<void>
}
```

Implementation example:

```typescript
class UserRepository implements Repository<User> {
  constructor(private db: Database) {}

  async findAll(filters?: UserFilters): Promise<User[]> {
    return this.db.users.findMany({ where: filters })
  }

  async findById(id: string): Promise<User | null> {
    return this.db.users.findUnique({ where: { id } })
  }

  async create(data: CreateUserDto): Promise<User> {
    return this.db.users.create({ data })
  }

  async update(id: string, data: UpdateUserDto): Promise<User> {
    return this.db.users.update({ where: { id }, data })
  }

  async delete(id: string): Promise<void> {
    await this.db.users.delete({ where: { id } })
  }
}
```
