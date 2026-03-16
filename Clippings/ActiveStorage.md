---
title: "ActiveStorage"
source: "https://api.rubyonrails.org/classes/ActiveStorage.html"
author:
published:
created: 2026-03-16
description: "Active Storage¶ ↑  Active Storage makes it simple to upload and reference files in cloud services like Amazon S3, or Google Cloud Storage, and attach those files to Active Records."
tags:
  - "clippings"
---
## Active Storage

Active Storage makes it simple to upload and reference files in cloud services like [Amazon S3](https://aws.amazon.com/s3/), or [Google Cloud Storage](https://cloud.google.com/storage/docs/), and attach those files to Active Records. Supports having one main service and mirrors in other services for redundancy. It also provides a disk service for testing or local deployments, but the focus is on cloud storage.

Files can be uploaded from the server to the cloud or directly from the client to the cloud.

Image files can furthermore be transformed using on-demand variants for quality, aspect ratio, size, or any other [MiniMagick](https://github.com/minimagick/minimagick) or [Vips](https://www.rubydoc.info/gems/ruby-vips/Vips/Image) supported transformation.

You can read more about Active Storage in the [Active Storage Overview](https://guides.rubyonrails.org/active_storage_overview.html) guide.

## Compared to other storage solutions

A key difference to how Active Storage works compared to other attachment solutions in Rails is through the use of built-in [Blob](https://github.com/rails/rails/blob/main/activestorage/app/models/active_storage/blob.rb) and [Attachment](https://github.com/rails/rails/blob/main/activestorage/app/models/active_storage/attachment.rb) models (backed by Active Record). This means existing application models do not need to be modified with additional columns to associate with files. Active Storage uses polymorphic associations via the [`Attachment`](https://api.rubyonrails.org/classes/ActiveStorage/Attachment.html) join model, which then connects to the actual [`Blob`](https://api.rubyonrails.org/classes/ActiveStorage/Blob.html).

[`Blob`](https://api.rubyonrails.org/classes/ActiveStorage/Blob.html) models store attachment metadata (filename, content-type, etc.), and their identifier key in the storage service. [`Blob`](https://api.rubyonrails.org/classes/ActiveStorage/Blob.html) models do not store the actual binary data. They are intended to be immutable in spirit. One file, one blob. You can associate the same blob with multiple application models as well. And if you want to do transformations of a given [`Blob`](https://api.rubyonrails.org/classes/ActiveStorage/Blob.html), the idea is that you’ll simply create a new one, rather than attempt to mutate the existing one (though of course you can delete the previous version later if you don’t need it).

## Installation

Run `bin/rails active_storage:install` to copy over [active\_storage](https://api.rubyonrails.org/files/activestorage/lib/active_storage_rb.html) migrations.

NOTE: If the task cannot be found, verify that `require "active_storage/engine"` is present in `config/application.rb`.

## Examples

One attachment:

```ruby
class User < ApplicationRecord
  # Associates an attachment and a blob. When the user is destroyed they are
  # purged by default (models destroyed, and resource files deleted).
  has_one_attached :avatar
end

# Attach an avatar to the user.
user.avatar.attach(io: File.open("/path/to/face.jpg"), filename: "face.jpg", content_type: "image/jpeg")

# Does the user have an avatar?
user.avatar.attached? # => true

# Synchronously destroy the avatar and actual resource files.
user.avatar.purge

# Destroy the associated models and actual resource files async, via Active Job.
user.avatar.purge_later

# Does the user have an avatar?
user.avatar.attached? # => false

# Generate a permanent URL for the blob that points to the application.
# Upon access, a redirect to the actual service endpoint is returned.
# This indirection decouples the public URL from the actual one, and
# allows for example mirroring attachments in different services for
# high-availability. The redirection has an HTTP expiration of 5 min.
url_for(user.avatar)

class AvatarsController < ApplicationController
  def update
    # params[:avatar] contains an ActionDispatch::Http::UploadedFile object
    Current.user.avatar.attach(params.require(:avatar))
    redirect_to Current.user
  end
end
```

Many attachments:

```ruby
class Message < ApplicationRecord
  has_many_attached :images
end
```
```xml
<%= form_with model: @message, local: true do |form| %>
  <%= form.text_field :title, placeholder: "Title" %><br>
  <%= form.textarea :content %><br><br>

  <%= form.file_field :images, multiple: true %><br>
  <%= form.submit %>
<% end %>
```
```ruby
class MessagesController < ApplicationController
  def index
    # Use the built-in with_attached_images scope to avoid N+1
    @messages = Message.all.with_attached_images
  end

  def create
    message = Message.create! params.expect(message: [ :title, :content, images: [] ])
    redirect_to message
  end

  def show
    @message = Message.find(params[:id])
  end
end
```

[`Variation`](https://api.rubyonrails.org/classes/ActiveStorage/Variation.html) of image attachment:

```xml
<%# Hitting the variant URL will lazy transform the original blob and then redirect to its new service location %>
<%= image_tag user.avatar.variant(resize_to_limit: [100, 100]) %>
```

## File serving strategies

Active Storage supports two ways to serve files: redirecting and proxying.

### Redirecting

Active Storage generates stable application URLs for files which, when accessed, redirect to signed, short-lived service URLs. This relieves application servers of the burden of serving file data. It is the default file serving strategy.

When the application is configured to proxy files by default, use the `rails_storage_redirect_path` and `_url` route helpers to redirect instead:

```ruby
<%= image_tag rails_storage_redirect_path(@user.avatar) %>
```

### Proxying

Optionally, files can be proxied instead. This means that your application servers will download file data from the storage service in response to requests. This can be useful for serving files from a CDN.

You can configure Active Storage to use proxying by default:

```ruby
# config/initializers/active_storage.rb
Rails.application.config.active_storage.resolve_model_to_route = :rails_storage_proxy
```

Or if you want to explicitly proxy specific attachments there are URL helpers you can use in the form of `rails_storage_proxy_path` and `rails_storage_proxy_url`.

```ruby
<%= image_tag rails_storage_proxy_path(@user.avatar) %>
```

## Direct uploads

Active Storage, with its included JavaScript library, supports uploading directly from the client to the cloud.

### Direct upload installation

1. Include the Active Storage JavaScript in your application's JavaScript bundle or reference it directly.
	Requiring directly without bundling through the asset pipeline in the application HTML with autostart:
	```xml
	<%= javascript_include_tag "activestorage" %>
	```
	Requiring via importmap-rails without bundling through the asset pipeline in the application HTML without autostart as ESM:
	```ruby
	# config/importmap.rb
	pin "@rails/activestorage", to: "activestorage.esm.js"
	```
	```xml
	<script type="module-shim">
	  import * as ActiveStorage from "@rails/activestorage"
	  ActiveStorage.start()
	</script>
	```
	Using the asset pipeline:
	```javascript
	//= require activestorage
	```
	Using the npm package:
	```xml
	import * as ActiveStorage from "@rails/activestorage"
	ActiveStorage.start()
	```
2. Annotate file inputs with the direct upload URL.
	```ruby
	<%= form.file_field :attachments, multiple: true, direct_upload: true %>
	```
3. Configure CORS on third-party storage services to allow direct upload requests.
4. That's it! Uploads begin upon form submission.

### Direct upload JavaScript events

| Event name | Event target | Event data (`event.detail`) | Description |
| --- | --- | --- | --- |
| `direct-uploads:start` | `<form>` | None | A form containing files for direct upload fields was submitted. |
| `direct-upload:initialize` | `<input>` | `{id, file}` | Dispatched for every file after form submission. |
| `direct-upload:start` | `<input>` | `{id, file}` | A direct upload is starting. |
| `direct-upload:before-blob-request` | `<input>` | `{id, file, xhr}` | Before making a request to your application for direct upload metadata. |
| `direct-upload:before-storage-request` | `<input>` | `{id, file, xhr}` | Before making a request to store a file. |
| `direct-upload:progress` | `<input>` | `{id, file, progress}` | As requests to store files progress. |
| `direct-upload:error` | `<input>` | `{id, file, error}` | An error occurred. An `alert` will display unless this event is canceled. |
| `direct-upload:end` | `<input>` | `{id, file}` | A direct upload has ended. |
| `direct-uploads:end` | `<form>` | None | All direct uploads have ended. |

Active Storage is released under the [MIT License](https://opensource.org/licenses/MIT).

## Support

API documentation is at:

- [api.rubyonrails.org](https://api.rubyonrails.org/)

Bug reports for the Ruby on Rails project can be filed here:

- [github.com/rails/rails/issues](https://github.com/rails/rails/issues)

Feature requests should be discussed on the rubyonrails-core forum here:

- [discuss.rubyonrails.org/c/rubyonrails-core](https://discuss.rubyonrails.org/c/rubyonrails-core)

- MODULE [ActiveStorage::Blobs](https://api.rubyonrails.org/classes/ActiveStorage/Blobs.html)
- MODULE [ActiveStorage::DisableSession](https://api.rubyonrails.org/classes/ActiveStorage/DisableSession.html)
- MODULE [ActiveStorage::Reflection](https://api.rubyonrails.org/classes/ActiveStorage/Reflection.html)
- MODULE [ActiveStorage::Representations](https://api.rubyonrails.org/classes/ActiveStorage/Representations.html)
- MODULE [ActiveStorage::SetCurrent](https://api.rubyonrails.org/classes/ActiveStorage/SetCurrent.html)
- MODULE [ActiveStorage::Streaming](https://api.rubyonrails.org/classes/ActiveStorage/Streaming.html)
- MODULE [ActiveStorage::Transformers](https://api.rubyonrails.org/classes/ActiveStorage/Transformers.html)
- MODULE [ActiveStorage::VERSION](https://api.rubyonrails.org/classes/ActiveStorage/VERSION.html)
- CLASS [ActiveStorage::AnalyzeJob](https://api.rubyonrails.org/classes/ActiveStorage/AnalyzeJob.html)
- CLASS [ActiveStorage::Analyzer](https://api.rubyonrails.org/classes/ActiveStorage/Analyzer.html)
- CLASS [ActiveStorage::Attached](https://api.rubyonrails.org/classes/ActiveStorage/Attached.html)
- CLASS [ActiveStorage::Attachment](https://api.rubyonrails.org/classes/ActiveStorage/Attachment.html)
- CLASS [ActiveStorage::BaseController](https://api.rubyonrails.org/classes/ActiveStorage/BaseController.html)
- CLASS [ActiveStorage::BaseJob](https://api.rubyonrails.org/classes/ActiveStorage/BaseJob.html)
- CLASS [ActiveStorage::Blob](https://api.rubyonrails.org/classes/ActiveStorage/Blob.html)
- CLASS [ActiveStorage::DirectUploadsController](https://api.rubyonrails.org/classes/ActiveStorage/DirectUploadsController.html)
- CLASS [ActiveStorage::DiskController](https://api.rubyonrails.org/classes/ActiveStorage/DiskController.html)
- CLASS [ActiveStorage::Error](https://api.rubyonrails.org/classes/ActiveStorage/Error.html)
- CLASS [ActiveStorage::FileNotFoundError](https://api.rubyonrails.org/classes/ActiveStorage/FileNotFoundError.html)
- CLASS [ActiveStorage::Filename](https://api.rubyonrails.org/classes/ActiveStorage/Filename.html)
- CLASS [ActiveStorage::FixtureSet](https://api.rubyonrails.org/classes/ActiveStorage/FixtureSet.html)
- CLASS [ActiveStorage::IntegrityError](https://api.rubyonrails.org/classes/ActiveStorage/IntegrityError.html)
- CLASS [ActiveStorage::InvariableError](https://api.rubyonrails.org/classes/ActiveStorage/InvariableError.html)
- CLASS [ActiveStorage::MirrorJob](https://api.rubyonrails.org/classes/ActiveStorage/MirrorJob.html)
- CLASS [ActiveStorage::Preview](https://api.rubyonrails.org/classes/ActiveStorage/Preview.html)
- CLASS [ActiveStorage::PreviewError](https://api.rubyonrails.org/classes/ActiveStorage/PreviewError.html)
- CLASS [ActiveStorage::PreviewImageJob](https://api.rubyonrails.org/classes/ActiveStorage/PreviewImageJob.html)
- CLASS [ActiveStorage::Previewer](https://api.rubyonrails.org/classes/ActiveStorage/Previewer.html)
- CLASS [ActiveStorage::PurgeJob](https://api.rubyonrails.org/classes/ActiveStorage/PurgeJob.html)
- CLASS [ActiveStorage::Service](https://api.rubyonrails.org/classes/ActiveStorage/Service.html)
- CLASS [ActiveStorage::TransformJob](https://api.rubyonrails.org/classes/ActiveStorage/TransformJob.html)
- CLASS [ActiveStorage::UnpreviewableError](https://api.rubyonrails.org/classes/ActiveStorage/UnpreviewableError.html)
- CLASS [ActiveStorage::UnrepresentableError](https://api.rubyonrails.org/classes/ActiveStorage/UnrepresentableError.html)
- CLASS [ActiveStorage::Variant](https://api.rubyonrails.org/classes/ActiveStorage/Variant.html)
- CLASS [ActiveStorage::VariantRecord](https://api.rubyonrails.org/classes/ActiveStorage/VariantRecord.html)
- CLASS [ActiveStorage::VariantWithRecord](https://api.rubyonrails.org/classes/ActiveStorage/VariantWithRecord.html)
- CLASS [ActiveStorage::Variation](https://api.rubyonrails.org/classes/ActiveStorage/Variation.html)

G

- [gem\_version](https://api.rubyonrails.org/classes/ActiveStorage.html#method-c-gem_version)

V

- [version](https://api.rubyonrails.org/classes/ActiveStorage.html#method-c-version)

## Class Public methods

### gem\_version()

Returns the currently loaded version of Active Storage as a `Gem::Version`.

Source: show | [on GitHub](https://github.com/rails/rails/blob/d7c8ae65b7045490965218a994c300aea8dbb079/activestorage/lib/active_storage/gem_version.rb#L5)

```
# File activestorage/lib/active_storage/gem_version.rb, line 5
def self.gem_version
  Gem::Version.new VERSION::STRING
end
```

### version()

Returns the currently loaded version of Active Storage as a `Gem::Version`.

Source: show | [on GitHub](https://github.com/rails/rails/blob/d7c8ae65b7045490965218a994c300aea8dbb079/activestorage/lib/active_storage/version.rb#L7)

```
# File activestorage/lib/active_storage/version.rb, line 7
def self.version
  gem_version
end
```