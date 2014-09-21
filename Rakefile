require "date"

namespace :post do
  desc "Creates a new blogg post"
  task new: :ask_for_title do
    file_name = "_posts/#{Date.today.to_s}-#{@title.as_file_name}"
    template = Template.new(title: @title)
    File.write(file_name, template.to_s)
    puts "Created file: #{file_name}"
  end

  task :ask_for_title do
    puts "What's the page title?"
    print " > "
    @title = Title.new($stdin.gets.chomp)
  end
end

class Title
  attr_accessor :value
  def initialize(value)
    @value = value
  end

  def to_s
    value.to_s
  end

  def as_file_name
    title = value.dup.downcase
    title.gsub! /\s+/, '-'
    "#{title}.md"
  end
end

class Template
  attr_accessor :layout, :title, :date
  def initialize(options = {})
    @layout     = options.fetch(:layout) { "post" }
    @title      = options.fetch(:title).to_s
    @date       = options.fetch(:date) { Date.today.to_s }
  end

  def to_s
    """---
layout:     #{layout}
title:      \"#{title}\"
date:       #{date}
---
"""
  end
end
