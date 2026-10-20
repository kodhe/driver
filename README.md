# Kodhe Driver Library - PSR-4 Compatible Driver System for CodeIgniter 3

A refactored, modular driver library system for CodeIgniter 3 with PSR-4 autoloading and modern design patterns.

## 📁 Structure

```
driver/
├── src/
│   ├── Contracts/
│   │   ├── DriverInterface.php          # Interface implemented by every driver
│   │   └── DriverLibraryInterface.php   # Interface for the parent library
│   ├── AbstractDriver.php               # Base abstract class for drivers
│   ├── DriverLibrary.php                # Base class for the parent library
│   ├── NullDriver.php                   # Null object pattern fallback
│   ├── Resolvers/
│   │   └── DriverFileResolver.php       # Resolves & loads driver files
│   ├── Registry/
│   │   └── DriverInstanceRegistry.php   # Registry pattern for caching instances
│   └── Traits/
│       ├── ConfigurableTrait.php        # Configuration handling
│       └── MagicCallTrait.php           # Magic method delegation
├── tests/
│   ├── DriverLibraryTest.php
│   ├── AbstractDriverTest.php
│   ├── Resolvers/
│   │   └── DriverFileResolverTest.php
│   └── Registry/
│       └── DriverInstanceRegistryTest.php
└── composer.json
```

## ✨ Features

- **PSR-4 Autoloading** - Namespace `Kodhe\Driver\`
- **PSR-12 Coding Standards** - Clean code style
- **Design Patterns**:
  - Registry Pattern (instance caching)
  - Factory Pattern (driver instantiation)
  - Proxy/Delegation Pattern (magic methods)
  - Template Method Pattern (abstract driver)
  - Null Object Pattern (fallback driver)
- **Dependency Injection** - Resolver and registry are injected via the constructor
- **Performance Optimized**:
  - Path caching for file lookup
  - Instance caching per request
  - Lazy loading drivers
- **100% Backward Compatible** - The CI3 API keeps working

## 🔧 Installation

```bash
composer require kodhe/driver:^2.0
```

Or add it to `composer.json`:

```json
{
  "require": {
    "kodhe/driver": "^2.0"
  }
}
```

## 📖 Usage

### Creating a Multi-Driver Library

```php
// application/libraries/Cache.php
class Cache extends \Kodhe\Driver\DriverLibrary
{
    protected $valid_drivers = ['file', 'apcu', 'redis', 'memcached'];
    
    public function __construct()
    {
        parent::__construct();
    }
}
```

### Creating a Driver

```php
// application/libraries/Cache/drivers/Cache_file.php
class Cache_file extends \Kodhe\Driver\AbstractDriver
{
    public function isSupported(): bool
    {
        return is_writable(APPPATH . 'cache/');
    }
    
    public function get($key)
    {
        // Implementation
    }
    
    public function save($key, $data, $ttl = 60)
    {
        // Implementation
    }
}
```

### Using It in a Controller

```php
class UserController extends CI_Controller
{
    public function __construct()
    {
        parent::__construct();
        $this->load->library('cache');
    }
    
    public function index()
    {
        // Access the driver via the magic getter (lazy loaded)
        $data = $this->cache->file->get('user_data');
        
        // Or load it explicitly
        $this->cache->load_driver('file');
        $data = $this->cache->file->get('user_data');
        
        // Check whether the driver is supported
        if ($this->cache->file->isSupported()) {
            $this->cache->file->save('key', 'value', 300);
        }
    }
}
```

## 🎯 Design Patterns

### Registry Pattern
```php
$registry = new DriverInstanceRegistry();
$registry->set('Cache', 'file', $driverInstance);
$cached = $registry->get('Cache', 'file'); // Returns cached instance
```

### Null Object Pattern
```php
// Safe fallback when a driver is not found
$driver = $this->isValidDriver($name) 
    ? $this->load_driver($name) 
    : new NullDriver();
```

### Dependency Injection
```php
$registry = new DriverInstanceRegistry();
$resolver = new DriverFileResolver();
$library = new MyLibrary($registry, $resolver);
```

## 🧪 Testing

```bash
composer test
composer test:coverage
```

## 📊 Comparison

| Area | Before (CI3 native) | After (Kodhe\Driver) |
|------|---------------------|------------------------|
| Structure | 2 monolithic classes | Modular (8+ components) |
| Namespace | None | PSR-4 `Kodhe\Driver\` |
| Testing | Hard (hardcoded paths) | Easy (DI + interfaces) |
| Caching | Manual per library | Built-in registry |
| Extensibility | Hard | Easy (implement an interface) |
| PSR compliance | None | PSR-4, PSR-12 |

## 🔒 Backward Compatibility

All features of the CI3 driver system keep working:
- ✅ `extends CI_Driver_Library` → `extends DriverLibrary`
- ✅ `extends CI_Driver` → `extends AbstractDriver`
- ✅ `$this->lib->driver_name->method()`
- ✅ `$this->parent->method()` from within a driver
- ✅ The `valid_drivers` and `lib_name` properties
- ✅ Magic methods `__get`, `__call`

Class aliases are available for full backward compatibility:

```php
// In compat.php (optional)
class_alias(\Kodhe\Driver\DriverLibrary::class, 'CI_Driver_Library');
class_alias(\Kodhe\Driver\AbstractDriver::class, 'CI_Driver');
```

## 📝 License

MIT License - see [LICENSE](LICENSE) for details.
