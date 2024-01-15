# PlanetScale Rails

[![Gem Version](https://badge.fury.io/rb/planetscale_rails.svg)](https://badge.fury.io/rb/planetscale_rails)

Rake tasks for easily running Rails migrations against PlanetScale database branches.

For information on how to connect your Rails app to PlanetScale, please [see our guide here](https://planetscale.com/docs/tutorials/connect-rails-app).

## Included rake tasks

The rake tasks allow you to use local MySQL for development. When you're ready to make a schema change, you can create a PlanetScale branch and run migrations
against it. See [usage](#usage) for details.

```
rake psdb:migrate                    # Migrate the database for current environment
rake psdb:rollback                   # Rollback primary database for current environment
rake psdb:schema:load                # Load the current schema into the database
```

## Installation

Add this line to your application's Gemfile:

```ruby
group :development do
  gem 'planetscale_rails'
end
```

And then execute in your terminal:

```
bundle install
```

Make sure you have the [`pscale` CLI installed](https://github.com/planetscale/cli#installation).

### Disable schema dump

Add the following to your `config/application.rb`.

```ruby
if ENV["DISABLE_SCHEMA_DUMP"]
  config.active_record.dump_schema_after_migration = false
end
```

This will allow the gem to disable schema dumping after running `psdb:migrate`.

## Usage

1. Using pscale, create a new branch. This command will create a local `.pscale.yml` file. You should add it to your `.gitignore`.

```
pscale branch switch my-new-branch-name --database my-db-name --create --wait
```

**Tip:** In your database settings. Enable "Automatically copy migration data." Select "Rails" as the migration framework. This will auto copy your `schema_migrations` table between branches.

2. Run your schema migrations against the branch.

```
bundle exec rails psdb:migrate
```

If you run multiple databases in your Rails app, you can specify the DB name.

```
bundle exec rails psdb:migrate:primary
```

If you make a mistake, you can use `bundle exec rails psdb:rollback` to rollback the changes on your PlanetScale branch.

3. Next, you can either open the Deploy Request via the UI. Or the CLI.

Via CLI is:
```
pscale deploy-request create database-name my-new-branch-name
```

4. To get your schema change to production, run the deploy request. Then, once it's complete, you can merge your code changes into your `main` branch in git and deploy your application code.

## Using PlanetScale deploy requests vs `psdb:migrate` directly in production.

PlanetScale's deploy requests [solve the schema change problem](https://planetscale.com/docs/learn/how-online-schema-change-tools-work). They make a normally high risk operation, safe. This is done by running your schema change using [Vitess's online schema change](https://vitess.io/docs/18.0/user-guides/schema-changes/) tools. Once the change is made, a deploy request is [also revertible without data loss](https://planetscale.com/blog/revert-a-migration-without-losing-data). None of this is possible when running `rails db:migrate` directly against your production database.

We recommend using GitHub Actions to automate the creation of PlanetScale branches and deploy requests. Then when you are ready to merge, you can run the deploy request before merging in your code.

**When not to use deploy requests**

If your application has minimal data and schema changes are a low risk event, then running `psdb:migrate` directly against production is perfectly fine. As your datasize grows and your application becomes busier, the risk of schema changes increase and we highly recommend using the deploy request flow. It's the best way available to safely migrate your schema.

## Usage with GitHub Actions

See the [GitHub Actions examples](actions-example.md) doc for ways to automate your schema migrations with PlanetScale + Actions.

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and the created tag, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/planetscale/planetscale_rails. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [code of conduct](https://github.com/planetscale/planetscale_rails/blob/main/CODE_OF_CONDUCT.md).

## License

The gem is available as open source under the terms of the [Apache 2.0 License](https://opensource.org/license/apache-2-0/).

## Code of Conduct

Everyone interacting in the PlanetScale Rails project's codebases, issue trackers, chat rooms and mailing lists is expected to follow the [code of conduct](https://github.com/planetscale/planetscale_rails/blob/main/CODE_OF_CONDUCT.md).
