require "bundler/gem_tasks"
require "jekyll"
require "listen"
require 'rake'
require 'date'
require 'yaml'


def listen_ignore_paths(base, options)
  [
    /_config\.ya?ml/,
    /_site/,
    /\.jekyll-metadata/
  ]
end

def listen_handler(base, options)
  site = Jekyll::Site.new(options)
  Jekyll::Command.process_site(site)
  proc do |modified, added, removed|
    t = Time.now
    c = modified + added + removed
    n = c.length
    relative_paths = c.map{ |p| Pathname.new(p).relative_path_from(base).to_s }
    print Jekyll.logger.message("Regenerating:", "#{relative_paths.join(", ")} changed... ")
    begin
      Jekyll::Command.process_site(site)
      puts "regenerated in #{Time.now - t} seconds."
    rescue => e
      puts "error:"
      Jekyll.logger.warn "Error:", e.message
      Jekyll.logger.warn "Error:", "Run jekyll build --trace for more information."
    end
  end
end

task :preview do
  base = Pathname.new('.').expand_path
  options = {
    "source"        => base.join('example').to_s,
    "destination"   => base.join('example/_site').to_s,
    "force_polling" => false,
    "serving"       => true,
    "theme"         => "jekyll-theme-basically-basic"
  }

  options = Jekyll.configuration(options)

  ENV["LISTEN_GEM_DEBUGGING"] = "1"
  listener = Listen.to(
    base.join("_includes"),
    base.join("_layouts"),
    base.join("_sass"),
    base.join("assets"),
    options["source"],
    :ignore => listen_ignore_paths(base, options),
    :force_polling => options['force_polling'],
    &(listen_handler(base, options))
  )

  begin
    listener.start
    Jekyll.logger.info "Auto-regeneration:", "enabled for '#{options["source"]}'"

    unless options['serving']
      trap("INT") do
        listener.stop
        puts "     Halting auto-regeneration."
        exit 0
      end

      loop { sleep 1000 }
    end
  rescue ThreadError
    # You pressed Ctrl-C, oh my!
  end

  Jekyll::Commands::Serve.process(options)
end

CONFIG = YAML.safe_load(File.read('_config.yml'))
USERNAME = CONFIG['github_username'] || ENV['GIT_NAME']
REPO = CONFIG['repo']
SSH_PASSWORD = ENV['SSH_PASSWORD'] || 'hunter2'
SSH_USERNAME = '128keaton.com'.freeze
SSH_HOST = 'my-host'.freeze
IMAGES_DIR = __dir__ + '/images/'

SOURCE_BRANCH = 'pre-publish'.freeze
DESTINATION_BRANCH = 'gh-pages'.freeze

def check_destination
    unless Dir.exist? CONFIG['destination']
        sh "git clone https://#{ENV['GIT_NAME']}:#{ENV['GITHUB_TOKEN']}@github.com/#{USERNAME}/#{REPO}.git #{CONFIG['destination']}"
    end
end

namespace :site do
  task :upload do
      puts 'Uploading site'
      sha = `git log`.match(/[a-z0-9]{40}/)[0]
      sh 'git add --all .'
      sh "git commit -m 'Updating to #{USERNAME}/#{REPO}@#{sha}.'"
      sh "git push --quiet origin #{SOURCE_BRANCH}"
      puts "Pushed updated branch #{SOURCE_BRANCH} to GitHub Pages"
  end

    task :correct_posts do
        puts 'Correcting blog posts'
        Dir.entries(__dir__ + '/_posts/').each do |file_name|
            next unless File.extname(file_name) == '.md' || File.extname(file_name) == '.markdown'
            text = File.read(__dir__ + '/_posts/' + file_name)
            fixed = text.gsub('](/images/', '](http://images.128keaton.com/')
            fixed = fixed.gsub('_images/', '')
            File.open(__dir__ + '/_posts/' + file_name, 'w') { |file| file.puts fixed }
        end
        Rake::Task["site:upload"].execute
    end
    task :upload_images do
        puts 'Uploading images..'
        options = { recursive: true, password: SSH_PASSWORD }
        Net::SCP.upload!(SSH_HOST, SSH_USERNAME, __dir__ + '/images/', '/home/12/128keaton.com/html/', options)
        puts 'All images have been upload and removed in ' + IMAGES_DIR
        FileUtils.rm_rf(Dir.glob(IMAGES_DIR + '*'))
    end
    desc 'Generate the site and push changes to remote origin'
    task :deploy do
        # Detect pull request
        if ENV['TRAVIS_PULL_REQUEST'].to_s.to_i > 0
            puts 'Pull request detected. Not proceeding with deploy.'
            exit
        end

        # Configure git if this is run in Travis CI
        if ENV['TRAVIS']
            sh "git config --global user.name '#{ENV['GIT_NAME']}'"
            sh "git config --global user.email '#{ENV['GIT_EMAIL']}'"
            sh 'git config --global push.default simple'
        end

        # Make sure destination folder exists as git repo
        check_destination
        sh 'git pull && git submodule init && git submodule update && git submodule status'
        sh "git checkout #{SOURCE_BRANCH}"
        Dir.chdir(CONFIG['destination']) { sh "git checkout #{DESTINATION_BRANCH}" }

        # Generate the site
        sh 'bundle exec jekyll build'

        # Commit and push to github
        sha = `git log`.match(/[a-z0-9]{40}/)[0]
        Dir.chdir(CONFIG['destination']) do
            sh 'git add --all .'
            sh "git commit -m 'Updating to #{USERNAME}/#{REPO}@#{sha}.'"
            sh "git push --quiet origin #{DESTINATION_BRANCH}"
            puts "Pushed updated branch #{DESTINATION_BRANCH} to GitHub Pages"
        end
    end
end
