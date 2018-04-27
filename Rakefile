module TempFixForRakeLastComment
  def last_comment
    last_description
  end
end
Rake::Application.send :include, TempFixForRakeLastComment

require "rspec/core/rake_task"

$LOAD_PATH.unshift File.expand_path("../lib", __FILE__)
require "moped/version"
require 'bundler/setup'
require 'gemfury'

Bundler::GemHelper.install_tasks

task :gem => :build
task :build do
  system "gem build moped.gemspec"
end

task :install => :build do
  system "sudo gem install moped-#{Moped::VERSION}.gem"
end

task :release => :build do
  system "git tag -a v#{Moped::VERSION} -m 'Tagging #{Moped::VERSION}'"
  system "git push --tags"
  system "gem push moped-#{Moped::VERSION}.gem"
  system "rm moped-#{Moped::VERSION}.gem"
end

RSpec::Core::RakeTask.new(:spec) do |spec|
  spec.pattern = "spec/**/*_spec.rb"
end

namespace :gemfury do
  desc 'Deploy new gems to Gemfury'
  task :deploy do
    client = Gemfury::Client.new(:user_api_key => ENV['GEMFURY_API_TOKEN'], :account => 'crashlytics')
    gemfury_version = {}
    GEMS = FileList["**/*.gem"]
    GEMS.each do |gem|
      # slug = current full id of the gem, like this: crashlytics-gemname-gemversion
      slug = File.basename(gem, '.gem')
      name, _, local_version = slug.rpartition('-')

      begin
        gemfury_version = client.versions(name).detect { |version| version['slug'] == slug }
      rescue Gemfury::NotFound, Faraday::Error::ParsingError
        gemfury_version = nil
      end

      if gemfury_version.nil?
        File.open(gem, 'r') do |f|
          client.push_gem(f)
        end
      end
    end
  end
end

task :default => :spec
