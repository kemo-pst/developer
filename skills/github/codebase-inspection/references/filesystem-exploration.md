# Filesystem Exploration Without Terminal

When `terminal` and `execute_code` are unavailable or require approval, use `search_files` and `read_file` to explore directory structures.

## Key Discovery: `file_glob` is Unreliable

The `file_glob` parameter in `search_files` may not actually filter results. Do not rely on it. Instead, use these patterns:

### Finding Top-Level Directory Structure

Search for a common file pattern using `target='content'`:

```python
# For Maven/Java projects - find all pom.xml files
search_files(pattern='<project', target='content', path='/path/to/repo')

# For any project - find all XML files
search_files(pattern='<', target='content', path='/path/to/repo', file_glob='*.xml')

# Find all Python files
search_files(pattern='^import ', target='content', path='/path/to/repo')
```

The results will show full paths, revealing the directory structure.

### Listing Files at a Specific Depth

Use `target='files'` with a broad search, then infer directories from paths:

```python
# Returns all files; directories can be inferred from path prefixes
search_files(pattern='^', target='files', path='/path/to/repo', limit=50)
```

### Reading Directory Contents

`read_file` cannot read directories - it returns an error. Use `search_files` instead.

## Practical Example

```python
# Step 1: Discover structure via pom.xml files
search_files(pattern='<project', target='content', path='/opt/data/pst-java-ch-spl')
# Returns: /opt/data/pst-java-ch-spl/pom.xml, /opt/data/pst-java-ch-spl/ch-spl-lib/pom.xml, etc.

# Step 2: Explore a specific module
search_files(pattern='^package ', target='content', path='/opt/data/pst-java-ch-spl/ch-spl-lib/src')
# Returns all Java files with their full paths

# Step 3: Read specific files
read_file(path='/opt/data/pst-java-ch-spl/pom.xml')
```

## Tips

- Use `limit` to avoid overwhelming results; paginate with `offset`
- Content search (`target='content'`) is more reliable than file search (`target='files'`) for discovering structure
- Search for project-specific patterns: `<project` (Maven), `import ` (Python), `package ` (Java), etc.
- The `output_mode='files_only'` flag can reduce noise in content searches
