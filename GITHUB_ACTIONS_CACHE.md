# GitHub Actions Caching Guide

## What is Caching in GitHub Actions?

Caching in GitHub Actions allows you to store and reuse files between workflow runs, significantly speeding up CI/CD pipelines by avoiding redundant downloads and builds.

### How It Works
- **First Run (Cache Miss)**: Downloads dependencies, saves them to cache
- **Subsequent Runs (Cache Hit)**: Restores dependencies from cache instantly
- **Result**: Faster builds, reduced bandwidth, lower costs

## How to Use Caching

### Basic Syntax
```yaml
- uses: actions/cache@v4
  with:
    path: |
      ~/.npm
      node_modules
    key: cache-name-${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      cache-name-${{ runner.os }}-
```

### Key Components

1. **`path`**: What to cache (directories/files)
2. **`key`**: Unique identifier (with dynamic values)
3. **`restore-keys`**: Fallback keys if exact match not found

## What to Cache

✅ **SHOULD Cache:**
- Package manager caches: `~/.npm`, `~/.cache/pip`, `~/.m2/repository`
- Dependencies: `node_modules`, `vendor`, `.venv`
- Build artifacts: compiled code, generated files
- Docker layers (with proper strategy)

❌ **Should NOT Cache:**
- Secrets or credentials
- Large files that change frequently
- Operating system artifacts
- Test results (use artifacts instead)
- Logs and debugging output
- Temporary files

## Controlling Cache Behavior

### Cache Duration
- **Default**: 7 days of inactivity
- **Maximum size**: 10 GB per repository
- **Automatic cleanup**: Older caches deleted when limit reached

### Cache Controls

1. **Key Strategy**:
   ```yaml
   key: v1-${{ runner.os }}-${{ hashFiles('**/lock-file') }}
   ```
   - Include version prefix (`v1-`) for manual invalidation
   - Use `runner.os` for OS-specific caches
   - Use `hashFiles()` for content-based keys

2. **Restore Keys** (fallback hierarchy):
   ```yaml
   restore-keys: |
     v1-${{ runner.os }}-
     v1-
   ```

3. **Conditional Caching**:
   ```yaml
   if: steps.cache.outputs.cache-hit != 'true'
   ```

## Cache Busting Techniques

### 1. **Change Key Prefix**
```yaml
key: v2-deps-${{ hashFiles('**/package.json') }}  # Changed from v1 to v2
```

### 2. **Manual Deletion**
- Go to repository → Actions → Caches
- Delete specific cache entries

### 3. **Workflow Dispatch**
```yaml
on:
  workflow_dispatch:
    inputs:
      clear-cache:
        description: 'Clear cache'
        type: boolean
```

### 4. **Use `${{ github.run_id }}`** (forces new cache every run)
```yaml
key: cache-${{ github.run_id }}  # Always unique
```

## Common Issues & Troubleshooting

### Issue 1: Cache Not Restoring
**Symptoms**: Always shows "Cache Miss"

**Solutions**:
- ✅ Verify `path` exists and contains files
- ✅ Check key consistency (OS, hash values)
- ✅ Ensure cache was successfully saved in previous run
- ✅ Check if cache expired (>7 days old)

### Issue 2: Stale Dependencies
**Symptoms**: Old versions loaded from cache

**Solutions**:
- ✅ Update key version: `v1-deps` → `v2-deps`
- ✅ Use `hashFiles()` on lock files
- ✅ Clear cache manually in GitHub UI

### Issue 3: Cache Size Exceeded
**Symptoms**: "Cache size exceeded" warning

**Solutions**:
- ✅ Cache only essential files
- ✅ Use `.gitignore` patterns in path
- ✅ Separate large caches into multiple keys
- ✅ Clean up old caches regularly

### Issue 4: Wrong Files Cached
**Symptoms**: Unexpected behavior after restore

**Solutions**:
- ✅ Review `path` configuration
- ✅ Use absolute paths or `${{ github.workspace }}`
- ✅ Check for conflicting cache entries

## Monitoring Cache Performance

### 1. **Check Cache Hit Rate**
```yaml
- name: Cache Statistics
  run: |
    echo "Cache Hit: ${{ steps.cache.outputs.cache-hit }}"
    echo "Cache Key: ${{ steps.cache.outputs.cache-primary-key }}"
```

### 2. **View in GitHub UI**
- Repository → Actions → Click workflow run
- Look for "Cache" annotations
- Check "Restore cache" and "Save cache" steps

### 3. **Measure Time Savings**
```yaml
- name: Measure build time
  run: |
    START=$SECONDS
    # ... build steps ...
    DURATION=$((SECONDS - START))
    echo "Build took $DURATION seconds"
```

### 4. **API Monitoring**
```bash
gh api repos/:owner/:repo/actions/caches
```

## Best Practices

1. **Use Composite Keys**
   ```yaml
   key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
   ```

2. **Layer Your Caches**
   ```yaml
   # OS-level cache
   key: v1-os-${{ runner.os }}
   
   # Dependency cache
   key: v1-deps-${{ hashFiles('**/lock.json') }}
   ```

3. **Cache Early, Fail Fast**
   - Place cache restore before expensive operations
   - Fail workflow if critical cache is missing

4. **Version Your Keys**
   - Increment version when cache schema changes
   - Example: `v1-deps` → `v2-deps`

5. **Document Cache Strategy**
   - Comment what's cached and why
   - Note cache invalidation triggers

## Example: Multi-Language Project

```yaml
- name: Cache dependencies
  uses: actions/cache@v4
  with:
    path: |
      ~/.npm
      ~/.cache/pip
      ~/.m2/repository
      node_modules
      .venv
      target
    key: ${{ runner.os }}-multi-${{ hashFiles('**/package-lock.json', '**/requirements.txt', '**/pom.xml') }}
    restore-keys: |
      ${{ runner.os }}-multi-
```

## Resources

- [GitHub Actions Cache Documentation](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows)
- [actions/cache Repository](https://github.com/actions/cache)
- [Cache API Reference](https://docs.github.com/en/rest/actions/cache)

---

**Cache Key Used in This Workflow**: `cache-61c3aa1`  
**Step Name**: `prime-cache-61c3aa1`
