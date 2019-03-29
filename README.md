# imgproxy.rb

[![CircleCI branch](https://img.shields.io/circleci/project/github/imgproxy/imgproxy.rb/master.svg?style=for-the-badge)](https://circleci.com/gh/imgproxy/imgproxy.rb) [![Gem](https://img.shields.io/gem/v/imgproxy.svg?style=for-the-badge)](https://rubygems.org/gems/imgproxy) [![rubydoc.org](https://img.shields.io/badge/rubydoc-reference-blue.svg?style=for-the-badge)](https://www.rubydoc.info/gems/imgproxy/)

Gem for [imgproxy](https://github.com/DarthSim/imgproxy) URLs generation.

[imgproxy](https://github.com/DarthSim/imgproxy) is a fast and secure standalone server for resizing and converting remote images. The main principles of imgproxy are simplicity, speed, and security.

## Installation

```
gem install imgproxy
```

or add it to your `Gemfile`:

```ruby
gem "imgproxy"
```

## Configuration

```ruby
# config/initializers/imgproxy.rb

Imgproxy.configure do |config|
  # imgproxy endpoint
  config.endpoint = "http://imgproxy.example.com"
  # hex-encoded signature key
  config.hex_key = "your_key"
  # hex-encoded signature salt
  config.hex_salt = "your_salt"
  # signature size. Defaults to 32
  config.signature_size = 5
  # use short processing option names (`rs` for `resize`, `g` for `gravity`, etc).
  # Defaults to true
  config.use_short_options = false
end
```

## Usage

```ruby
Imgproxy.url_for(
  "http://images.example.com/images/image.jpg",
  width: 500,
  height: 400,
  resizing_type: :fill,
  sharpen: 0.5
)
# => http://imgproxy.example.com/2tjGMpWqjO/rs:fill:500:400/sh:0.5/plain/http://images.example.com/images/image.jpg
```

You can reuse processing options by using `Imgproxy::Builder`:

```ruby
builder = Imgproxy::Builder.new(
  width: 500,
  height: 400,
  resizing_type: :fill,
  sharpen: 0.5
)

builder.url_for("http://images.example.com/images/image1.jpg")
builder.url_for("http://images.example.com/images/image2.jpg")
```

Available options are:

* `resizing_type`
* `width`
* `height`
* `dpr`
* `enlarge`
* `extend`
* `gravity`
* `gravity_x`
* `gravity_y`
* `quality`
* `background`
* `blur`
* `sharpen`
* `watermark_opacity`
* `watermark_position`
* `watermark_x_offset`
* `watermark_y_offset`
* `watermark_scale`
* `preset`
* `cachebuster`
* `format`
* `use_short_options`

_See [imgproxy URL format guide](https://github.com/DarthSim/imgproxy/blob/master/docs/generating_the_url_advanced.md) for more info_

### Using with ActiveStorage

If you use [ActiveStorage](https://guides.rubyonrails.org/active_storage_overview.html), you can configure imgproxy gem to work with ActiveStorage attachments:

```ruby
Imgproxy.extend_active_storage

# Now you can use ActiveStorage attachment as a source URL
Imgproxy.url_for(user.avatar, width: 250, height: 250)
# or you can use #imgproxy_url method of an attachment
user.avatar.imgproxy_url(width: 250, height: 250)
```

If you configured both your imgproxy server and ActiveStorage for working with Amazon S3, you may want to use short and beautiful `s3://...` source URLs instead of long ones generated by Rails:

```ruby
Imgproxy.extend_active_storage(use_s3: true)
```

You can do the same for Google Cloud Storage:

```ruby
# ActiveStorage hides GCS config in private, so we have to provide GCS bucket name
Imgproxy.extend_active_storage(use_gcs: true, gcs_bucket: "my_bucket")
```

### Using with Shrine

If you use [Shrine](https://shrinerb.com/), you can configure imgproxy gem to work with `Shrine::UploadedFile`:

```ruby
Imgproxy.extend_shrine

# Now you can use Shrine::UploadedFile as a source URL
Imgproxy.url_for(user.avatar, width: 250, height: 250)
# or you can use #imgproxy_url method of an Shrine::UploadedFile
user.avatar.imgproxy_url(width: 250, height: 250)
```

**Note:** If you use `Shrine::Storage::FileSystem` as a storage, uploaded file URLs won't include host and imgproxy server won't have access to them. To fix this, initialize `Shrine::Storage::FileSystem` with `host` option:

```ruby
Shrine.storages = {
  store: Shrine::Storage::FileSystem.new("public", host: "http://your-host.test")
}
```

Or you can launch your imgproxy server with `IMGPROXY_BASE_URL` setting:

```
IMGPROXY_BASE_URL="http://your-host.test" imgproxy
```

If you configured both your imgproxy server and ActiveStorage for working with Amazon S3, you may want to use short and beautiful `s3://...` source URLs:

```ruby
Imgproxy.extend_shrine(use_s3: true)
```

### URL adapters

By default, `Imgproxy.url_for` accepts only `String` or `URI` as source URL, but you can extend this by using URL adapters. URL adapter is a simple class that implements `applicable?` and `url` methods. See the example below:

```ruby
class MyItemAdapter
  # `applicable?` checks if the adapter can extract source URL from the provided object
  def applicable?(item)
    item.is_a? MyItem
  end

  # `url` extracts source URL from the provided object
  def url(item)
    item.image_url
  end
end

Imgproxy.configure do |config|
  config.url_adapters.add MyItemAdapter.new
end
```

**Note:** `Imgproxy` will use the first applicable URL adapter. If you need to add your adapter to the beginning of the list, use `prepend` method instead of `add`.

imgproxy gem provides adapters for ActiveStorage and Shrine that are automatically added by `Imgproxy.extend_active_storage` and `Imgproxy.extend_shrine`.

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/imgproxy/imgproxy.rb.

## License
The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).
