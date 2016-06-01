#!/usr/bin/env rake
require 'fileutils'
require 'haml'
require 'toml'
require 'json'

FORMATS = Dir.glob("src/*.haml").map{|f| File.basename(f,".haml")}.freeze

task :default => [:html, :pdf]

desc "Convert to html"
task :html do
  resume = JSON.parse(TOML.load_file('data/resume.toml').to_json, object_class: OpenStruct)

  FORMATS.each do |fmt|
    File.open("#{fmt}.html", "w") do |f|
      f << to_html(fmt, resume)
    end
  end
end

desc "Convert to pdf"
task :pdf do
  FORMATS.each do |fmt|
    next unless File.exist?("#{fmt}.html")
    to_pdf(fmt)
  end
end

desc "Clean up generated files"
task :clean do
  Dir['*.html', '*.pdf'].each do |file|
    FileUtils.safe_unlink file
  end
end

task :publish do
  abort "Uncommitted changes!" unless sh("git diff-index --quiet HEAD --")

  Rake::Task['default'].invoke
  sh("git commit -am 'Update html and pdf'")
  sh("git tag -f #{Time.now.strftime("%Y.%m.%d")}")
  sh("git push && git push --tags")
end

def to_html(fmt, resume)
  Haml::Engine.new(File.read("src/#{fmt}.haml")).render(Object.new, resume: resume)
end

def to_pdf(fmt)
  sh("wkhtmltopdf -s letter #{fmt}.html --print-media-type #{fmt}.pdf")
end
