# Laravel Framework 11.32.0 - Artisan Commands

## Usage
```
command [options] [arguments]
```

### Options:
- **`-h, --help`**: Display help for the given command. When no command is given, display help for the list command.
- **`-q, --quiet`**: Do not output any message.
- **`-V, --version`**: Display the application version.
- **`--ansi|--no-ansi`**: Force (or disable) ANSI output.
- **`-n, --no-interaction`**: Do not ask any interactive question.
- **`--env[=ENV]`**: Specify the environment for the command.
- **`-v|vv|vvv, --verbose`**: Increase verbosity:
  - `1`: Normal output.
  - `2`: More verbose output.
  - `3`: Debug output.

---

## Available Commands

### General
- **`about`**: Display basic information about your application.
- **`clear-compiled`**: Remove the compiled class file.
- **`completion`**: Dump the shell completion script.
- **`db`**: Start a new database CLI session.
- **`docs`**: Access the Laravel documentation.
- **`down`**: Put the application into maintenance/demo mode.
- **`env`**: Display the current framework environment.
- **`help`**: Display help for a command.
- **`inspire`**: Display an inspiring quote.
- **`list`**: List all available commands.
- **`migrate`**: Run the database migrations.
- **`optimize`**: Cache framework bootstrap, configuration, and metadata for performance.
- **`pail`**: Tails the application logs.
- **`serve`**: Serve the application on the PHP development server.
- **`test`**: Run the application tests.
- **`tinker`**: Interact with your application.
- **`up`**: Bring the application out of maintenance mode.

---

### Auth
- **`auth:clear-resets`**: Flush expired password reset tokens.

---

### Cache
- **`cache:clear`**: Flush the application cache.
- **`cache:forget`**: Remove an item from the cache.
- **`cache:prune-stale-tags`**: Prune stale cache tags from the cache (Redis only).

---

### Channel
- **`channel:list`**: List all registered private broadcast channels.

---

### Config
- **`config:cache`**: Create a cache file for faster configuration loading.
- **`config:clear`**: Remove the configuration cache file.
- **`config:publish`**: Publish configuration files to your application.
- **`config:show`**: Display values for a configuration file or key.

---

### Database
- **`db:monitor`**: Monitor the number of connections on the specified database.
- **`db:seed`**: Seed the database with records.
- **`db:show`**: Display information about the given database.
- **`db:table`**: Display information about the given database table.
- **`db:wipe`**: Drop all tables, views, and types.

---

### Environment
- **`env:decrypt`**: Decrypt an environment file.
- **`env:encrypt`**: Encrypt an environment file.

---

### Event
- **`event:cache`**: Discover and cache the application's events and listeners.
- **`event:clear`**: Clear all cached events and listeners.
- **`event:list`**: List the application's events and listeners.

---

### Fortify
- **`fortify:install`**: Install all of the Fortify resources.

---

### Installation
- **`install:api`**: Create an API routes file and install Laravel Sanctum or Laravel Passport.
- **`install:broadcasting`**: Create a broadcasting channel routes file.

---

### Jetstream
- **`jetstream:install`**: Install the Jetstream components and resources.

---

### Key
- **`key:generate`**: Set the application key.

---

### Language
- **`lang:publish`**: Publish all language files available for customization.

---

### Livewire
- **`livewire:attribute`**: Create a new Livewire attribute class.
- **`livewire:configure-s3-upload-cleanup`**: Configure temporary file uploads to auto-clean S3 files older than 24 hours.
- **`livewire:copy`**: Copy a Livewire component.
- **`livewire:delete`**: Delete a Livewire component.
- **`livewire:form`**: Create a new Livewire form class.
- **`livewire:layout`**: Create a new app layout file.
- **`livewire:make`**: Create a new Livewire component.
- **`livewire:move`**: Move a Livewire component.
- **`livewire:publish`**: Publish Livewire configuration.
- **`livewire:stubs`**: Publish Livewire stubs.
- **`livewire:upgrade`**: Interactive upgrade helper for migrating from v2 to v3.

---

### Make
- **`make:cache-table`**: Create a migration for the cache database table.
- **`make:cast`**: Create a new custom Eloquent cast class.
- **`make:channel`**: Create a new channel class.
- **`make:class`**: Create a new class.
- **`make:command`**: Create a new Artisan command.
- **`make:component`**: Create a new view component class.
- **`make:controller`**: Create a new controller class.
- **`make:enum`**: Create a new enum.
- **`make:event`**: Create a new event class.
- **`make:exception`**: Create a new custom exception class.
- **`make:factory`**: Create a new model factory.
- **`make:interface`**: Create a new interface.
- **`make:job`**: Create a new job class.
- **`make:job-middleware`**: Create a new job middleware class.
- **`make:listener`**: Create a new event listener class.
- **`make:livewire`**: Create a new Livewire component.
- **`make:mail`**: Create a new email class.
- **`make:middleware`**: Create a new HTTP middleware class.
- **`make:migration`**: Create a new migration file.
- **`make:model`**: Create a new Eloquent model class.
- **`make:notification`**: Create a new notification class.
- **`make:notifications-table`**: Create a migration for the notifications table.
- **`make:observer`**: Create a new observer class.
- **`make:policy`**: Create a new policy class.
- **`make:provider`**: Create a new service provider class.
- **`make:queue-batches-table`**: Create a migration for the batches database table.
- **`make:queue-failed-table`**: Create a migration for the failed queue jobs database table.
- **`make:queue-table`**: Create a migration for the queue jobs database table.
- **`make:request`**: Create a new form request class.
- **`make:resource`**: Create a new resource.
- **`make:rule`**: Create a new validation rule.
- **`make:scope`**: Create a new scope class.
- **`make:seeder`**: Create a new seeder class.
- **`make:session-table`**: Create a migration for the session database table.
- **`make:test`**: Create a new test class.
- **`make:trait`**: Create a new trait.
- **`make:view`**: Create a new view.

---

### Migrate
- **`migrate:fresh`**: Drop all tables and re-run all migrations.
- **`migrate:install`**: Create the migration repository.
- **`migrate:refresh`**: Reset and re-run all migrations.
- **`migrate:reset`**: Rollback all database migrations.
- **`migrate:rollback`**: Rollback the last database migration.
- **`migrate:status`**: Show the status of each migration.

---

### Model
- **`model:prune`**: Prune models no longer needed.
- **`model:show`**: Show information about an Eloquent model.

---

### Optimize
- **`optimize:clear`**: Remove the cached bootstrap files.

---

### Permission
- **`permission:cache-reset`**: Reset the permission cache.
- **`permission:create-permission`**: Create a permission.
- **`permission:create-role`**: Create a role.
- **`permission:setup-teams`**: Set up the teams feature by generating migrations.
- **`permission:show`**: Show a table of roles and permissions.

---

### Queue
- **`queue:clear`**: Delete all jobs from the specified queue.
- **`queue:failed`**: List all failed queue jobs.
- **`queue:flush`**: Flush all failed queue jobs.
- **`queue:forget`**

: Delete a failed queue job.
- **`queue:listen`**: Listen to a given queue.
- **`queue:monitor`**: Monitor the size of specified queues.
- **`queue:prune-batches`**: Prune stale entries from batches.
- **`queue:prune-failed`**: Prune stale failed job entries.
- **`queue:restart`**: Restart queue worker daemons.
- **`queue:retry`**: Retry a failed queue job.
- **`queue:retry-batch`**: Retry failed jobs for a batch.
- **`queue:work`**: Start processing jobs as a daemon.

---

### Route
- **`route:cache`**: Create a route cache file for faster registration.
- **`route:clear`**: Remove the route cache file.
- **`route:list`**: List all registered routes.

---

### Storage
- **`storage:link`**: Create symbolic links for storage.
- **`storage:unlink`**: Delete existing symbolic links.

---

### Sail
- **`sail:add`**: Add a service to an existing Sail installation.
- **`sail:install`**: Install Laravel Sail's default Docker Compose file.
- **`sail:publish`**: Publish the Laravel Sail Docker files.

---

### Sanctum
- **`sanctum:prune-expired`**: Prune tokens that have expired for more than the specified number of hours.

---

### Schedule
- **`schedule:clear-cache`**: Delete the cached mutex files created by the scheduler.
- **`schedule:interrupt`**: Interrupt the current schedule run.
- **`schedule:list`**: List all scheduled tasks.
- **`schedule:run`**: Run the scheduled commands.
- **`schedule:test`**: Run a scheduled command for testing purposes.
- **`schedule:work`**: Start the schedule worker.

---

### Schema
- **`schema:dump`**: Dump the given database schema.

---

### Stub
- **`stub:publish`**: Publish all stubs that are available for customization.

---

### User
- **`user:make-admin`**: Assign the admin role to a user.

---

### Vendor
- **`vendor:publish`**: Publish any publishable assets from vendor packages.

---

### View
- **`view:cache`**: Compile all of the application's Blade templates.
- **`view:clear`**: Clear all compiled view files.
