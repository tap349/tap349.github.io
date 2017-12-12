---
layout: post
title: Rails - Testing
date: 2017-12-12 18:45:59 +0300
access: public
comments: true
categories: [rails]
---

<!-- more -->

* TOC
{:toc}
<hr>

testing operations that receive params from controller
------------------------------------------------------

controller:

```ruby
# or else ditch strong paramaters altogether in
# favour of dry-validation schema inside operation
def create_params
  # all keys and values in resulting hash are strings
  params.require(:user).permit(:name, :age).to_unsafe_h
end
```

operation spec:

```ruby
describe Users::Create do
  subject(:call) { described_class.(params: params) }
  # all keys and values must be strings here too
  let(:params) { { 'name' => 'Jane', 'age' => '20' } }

  it do
    user = call
    expect(user).to have_attributes(
      name: params['name'],
      age: params['age'].to_i
    )
  end
end
```
