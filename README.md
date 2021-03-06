# data-guard
![](https://github.com/cdinopol/data-guard/workflows/Tests/badge.svg?branch=master)
[![Latest Stable Version](http://poser.pugx.org/cdinopol/data-guard/v)](https://packagist.org/packages/cdinopol/data-guard)
[![License](http://poser.pugx.org/cdinopol/data-guard/license)](https://packagist.org/packages/cdinopol/data-guard)

Filter out data from an array of a given conditions

## Installation
```sh
composer require cdinopol/data-guard
```

## Usage
```
DataGuard::protect(array $data, string $resource, array $conditions, $mask (optional));
```

### Data
- your data array (preferably an associative array)

### Resource
- string (example format: `'orders[]|order:line_items[]:sku'`)
- this is the key point of data to be processed.
- `|` - key split, keys to match on the same level.
- `:` - key separator, hierarchy of keys to match from root to child.
- `[]` - array indicator, DataGuard will look inside each of the values instead of directly looking for the next key.

### Conditions
- Conditions can be formatted into 3 types:
1. `"*"` - means it will match all from the given resource.
2. `[[operator, value]]` - this will match the given resource directly to the search value.
3. `[[search_resource, operator, value]]` - instead of matching the given resource directly, you can pass another resource (same formatting as resource) as the first index of condition to match against the operator+value. search_resource will be searched through and matched, but the process point will still be on the given resource.
- All conditions in an array must match to be considered positive. Tip: Running OR condition means running DataGuard with another set conditions

### Condition Operators
```
1. =     : equals
2. !=    : not equals
3. in    : in array
4. !in   : not in array
5. >     : greater than
6. <     : less than
7. regex : Regular Expression; condition value must be a proper expression
```

### Mask
- any value, optional.
- will replace the resource value to this instead of removing it.

## Usage
```php
use Cdinopol\DataGuard\DataGuard;

$data = [
    'hero' => [
        'name' => 'Thor',
        'profile' => [
            'address' => [
                'city' => 'Asgard',
                'country' => 'Asgard',
            ],
        ],

    ],
    'villain' => [
        'name' => 'Loki',
        'profile' => [
            'address' => [
                'city' => 'Asgard',
                'country' => 'Asgard',
            ],
        ],
    ],
    'others' => [
        [
            'name' => 'John',
            'profile' => [
                'address' => [
                    'city' => 'Asgard',
                    'country' => 'Asgard',
                ],
            ],
        ],
        [
            'name' => 'Doe',
            'profile' => [
                'address' => [
                    'city' => 'New York',
                    'country' => 'USA',
                ],
            ],
        ],
        [
            'name' => 'Carl',
            'profile' => [
                'addresses' => [
                    [
                        'city' => 'Chicago',
                        'country' => 'USA',
                    ],
                    [
                        'city' => 'Asgard',
                        'country' => 'Asgard',
                    ],
                ],
            ],
        ],
    ],
];

// Hides profile if city = Asgard
$resource = 'heroes[]|hero|villain|others[]:profile';
$conditions = [['address|addresses[]:city', '=', 'Asgard']];
$protectedData = DataGuard::protect($data, $resource, $conditions);

print_r($protectedData);
# Result:
[
    'hero' => [
        'name' => 'Thor',
    ],
    'villain' => [
        'name' => 'Loki',
    ],
    'others' => [
        [
            'name' => 'John',
        ],
        [
            'name' => 'Doe',
            'profile' => [
                'address' => [
                    'city' => 'New York',
                    'country' => 'USA',
                ],
            ],
        ],
        [
            'name' => 'Carl',
        ],
    ],
];
```

Please check the [unit test](tests/DataGuardTest.php) for more usage examples.

## License
The MIT License (MIT). Please see [License File](LICENSE) for more information.
