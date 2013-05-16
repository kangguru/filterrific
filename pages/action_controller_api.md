---
layout: default
---

<div class="page-header">
  <h2>Controller API</h2>
</div>

{% include project_navigation.html %}

The Filterrific ActionController API has the following functions:

* Initialize filter settings from params, persistence or defaults.
* Execute the ActiveRecord query to get the list of filtered records.
* Persist the current filter settings.
* Send the ActiveRecord collection to the view for rendering.
* Reset the filter settings.



### Index action

Filterrific lives in the UserController's index action. Please see the code
below for notes and implementation:

```ruby
# app/controllers/users_controller.rb
class UsersController < ApplicationController

  def index
    # Initialize the filter settings from the following data:
    # 1. Request params from filter form submission
    # 2. Params persisted in session
    # 3. Defaults defined in the User model
    @filterrific = Filterrific.new(
      User,
      params[:filterrific] || session[:filterrific_users]
    )

    # Get an ActiveRecord relation for all users that match the filter settings.
    # You can paginate with will_paginate or kaminari.
    # NOTE: filterrific_find returns an ActiveRecord Relation that can be
    # chained with other scopes to further narrow down the scope of the list,
    # e.g., to apply permissions or to hard coded exclude certain types of records.
    @users = User.filterrific_find(@filterrific).page(params[:page])

    # Persist the current filter settings in the session as a plain old Hash.
    session[:filterrific_users] = @filterrific.to_hash

    # Respond to html for initial page load and to js for AJAX filter updates.
    respond_to do |format|
      format.html
      format.js
    end
  end

  ...

end
```



### Reset filter settings

A useful feature is a "Reset filters" link that restores the default filter
settings. To do so, just add a `reset_filterrific` action to the `UsersController`
and add a route for it.

```ruby
# app/controllers/users_controller.rb
class UsersController < ApplicationController

  ...

  def reset_filterrific
    # Clear session persistence
    session[:filterrific_users] = nil
    # Redirect back to the index action for default filter settings.
    redirect_to :action => :index
  end

end
```

Add a collection route to your `user` resources:

```ruby
# config/routes.rb
resources :user do
  collection do
    get :reset_filterrific
  end
end
```

Please see the View API for details on the "Reset filters" button.



<div class="pull-right">
  <a href="/pages/action_view_api.html" class='btn btn-success'>Learn about the View API &rarr;</a>
</div>
