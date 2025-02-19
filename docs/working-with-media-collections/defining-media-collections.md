---
title: Defining media collections
weight: 2
---

A media collection can be more than [just a name to group files](/laravel-medialibrary/v7/working-with-media-collections/simple-media-collections). By defining a media collection in your model you can add certain behaviour collections.

To get started with media collections add a function called `registerMediaCollections` to [your prepared model](/laravel-medialibrary/v7/basic-usage/preparing-your-model). Inside that function you can use `addMediaCollection` to start  a media collection.

```php
// in your model

public function registerMediaCollections()
{
    $this->addMediaCollection('my-collection');
        //add options
        ...

    // you can define as many collections as needed
    $this->addMediaCollection('my-other-collection');
        //add options
        ...
}
```

## Defining a fallback url or path

If your media collection does not contain any items, calling `getFirstMediaUrl` or `getFirstMediaPath` will return `null`. You can change this by setting a fallback url and/or path using `useFallbackUrl` and `useFallbackPath`.

```php
use Spatie\MediaLibrary\File;
...
public function registerMediaCollections()
{
    $this
        ->addMediaCollection('avatars')
        ->useFallbackUrl('/images/anonymous-user.jpg')
        ->useFallbackPath(public_path('/images/anonymous-user.jpg'));
}
```

## Only allow certain files in a collection

You can pass a callback to `acceptsFile` that will check if a file is allowed into the collection. In this example we only accept `jpeg` files.

```php
use Spatie\MediaLibrary\File;
...
public function registerMediaCollections()
{
    $this
        ->addMediaCollection('only-jpegs-please')
        ->acceptsFile(function (File $file) {
            return $file->mimeType === 'image/jpeg';
        });
}
```

This will succeed:

```php
$yourModel->addMedia('beautiful.jpg')->toMediaCollection('only-jpegs-please');
```

This will throw a `Spatie\MediaLibrary\Exceptions\FileCannotBeAdded\FileUnacceptableForCollection` exception:

```php
$yourModel->addMedia('ugly.ppt')->toMediaCollection('only-jpegs-please');
```

## Using a specific disk

You can ensure that files added to a collection are automatically added to a certain disk.

```php
// in your model

public function registerMediaCollections()
{
    $this
       ->addMediaCollection('big-files')
       ->useDisk('s3');
}
```

When adding a file to `big-files` it will be stored on the `s3` disk.

```php
$yourModel->addMedia($pathToFile)->toMediaCollection('big-files');
```

You can still specify the disk name manually when adding media. In this example the file will be stored on `alternative-disk` instead of `s3`.

```php
$yourModel->addMedia($pathToFile)->toMediaCollection('big-files', 'alternative-disk');
```

## Single file collections

If you want a collection to hold only one file you can use `singleFile` on the collection. A good use case for this would be an avatar collection on a `User` model. In most cases you'd want to have a user to only have one `avatar`.

```php
// in your model

public function registerMediaCollections()
{
    $this
        ->addMediaCollection('avatar')
        ->singleFile();
}
```

The first time you add a file to the collection it will be stored as usual.

```php
$yourModel->addMedia($pathToImage)->toMediaCollection('avatar');
$yourModel->getMedia('avatar')->count(); // returns 1
$yourModel->getFirstMediaUrl('avatar'); // will return an url to the `$pathToImage` file
```

When adding another file to a single file collection the first one will be deleted.

```php
// this will remove other files in the collection
$yourModel->addMedia($anotherPathToImage)->toMediaCollection('avatar');
$yourModel->getMedia('avatar')->count(); // returns 1
$yourModel->getFirstMediaUrl('avatar'); // will return an url to the `$anotherPathToImage` file
```

## Limited file collections

Whenever you want to limit the amount of files inside a collection you can use the `onlyKeepLatest(n)` method. Whenever you add a file to a collection and exceed the given limit, Medialibrary will delete the oldest file(s) and keep the collection size at `n`.

```php
// in your model

public function registerMediaCollections()
{
    $this
        ->addMediaCollection('limited-collection')
        ->onlyKeepLatest(3);
}
```

For the first 3 files, nothing strange happens. The files get added to the collection and the collection now holds all 3 files. Whenever you decide to add a 4th file, Medialibrary deletes the first file and keeps the latest 3.

```php
$yourModel->addMedia($firstFile)->toMediaCollection('limited-collection');
$yourModel->getMedia('avatar')->count(); // returns 1
$yourModel->addMedia($secondFile)->toMediaCollection('limited-collection');
$yourModel->getMedia('avatar')->count(); // returns 2
$yourModel->addMedia($thirdFile)->toMediaCollection('limited-collection');
$yourModel->getMedia('avatar')->count(); // returns 3
$yourModel->addMedia($fourthFile)->toMediaCollection('limited-collection');
$yourModel->getMedia('avatar')->count(); // returns 3
$yourModel->getFirstMediaUrl('avatar'); // will return an url to the `$secondFile` file
```

## Registering media conversions

It's recommended that your first read the section on [converting images](/laravel-medialibrary/v7/converting-images/defining-conversions) before reading the following paragraphs.

Normally image conversions are registered inside the `registerMediaConversions` function on your model. However, images conversions can also be registered inside media collections.

```php
use Spatie\MediaLibrary\Models\Media;

public function registerMediaCollections()
{
    $this
        ->addMediaCollection('my-collection')
        ->registerMediaConversions(function (Media $media) {
            $this
                ->addMediaConversion('thumb')
                ->width(100)
                ->height(100);
        });
}
```

When adding an image to `my-collection` a thumbnail that fits inside 100x100 will be created.

```php
$yourModel->add($pathToImage)->toMediaCollection('my-collection');

$yourModel->getFirstMediaUrl('thumb') // returns an url to a 100x100 version of the added image.
```

Take a look at the [defining conversions section](/laravel-medialibrary/v7/converting-images/defining-conversions) to learn all the functions you can tack on to `addMediaConversion`.
