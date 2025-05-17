require "middleman-gh-pages"

require "yaml"
require 'active_support/inflector'
require 'active_support/core_ext/string/inflections'
require 'active_support/core_ext/time'

BASE_URL = "https://trug.wawer.it"

def generate_actor(talk)
  return unless talk["full_name"]
  filename = ActiveSupport::Inflector.transliterate(talk["full_name"]).parameterize
  path = "source/actors/#{filename}.json"
  return if File.exist?(path)

  data = {
    "@context" => "https://www.w3.org/ns/activitystreams",
    "id" => "#{BASE_URL}/actors/#{filename}.json",
    "type" => "User",
    "name" => talk["full_name"]
  }
  data["url"] = talk["home_page"] if talk["home_page"]

  File.write(path, JSON.pretty_generate(data))
end

def generate_talk(talk, event)
  return unless talk["title"]

  actor_filename = talk["full_name"] ? ActiveSupport::Inflector.transliterate(talk["full_name"]).parameterize : nil
  filename = "#{event["date"]}-#{(event["talks"].index(talk) + 1).to_s.rjust(3, "0")}"
  path = "source/talks/#{filename}.json"

  time = Time.parse("#{event["date"]} 18:00")
  time_with_timezone = time.in_time_zone("Europe/Warsaw")

  data = {
    "@context" => "https://www.w3.org/ns/activitystreams",
    "id" => "#{BASE_URL}/talks/#{filename}.json",
    "type" => "Note",
    "name" => talk["title"],
    "attributedTo" => nil,
    "inReplyTo" => "#{BASE_URL}/events/#{event["date"]}.json",
    "attachment" => [],
    "published" => time_with_timezone.iso8601
  }

  data["attributedTo"] = "#{BASE_URL}/actors/#{actor_filename}.json" if actor_filename

  case talk["video_provider"]
  when "vimeo"
    data["attachment"] << {
      "type" => "Video",
      "url" => "https://player.vimeo.com/video/#{talk["video_id"]}",
      "name" => "Wideo",
    }
  when "youtube"
    data["attachment"] << {
      "type" => "Video",
      "url" => "https://www.youtube.com/watch?v=#{talk["video_id"]}",
      "name" => "Wideo",
    }
  end

  if talk["slides"].to_s != ""
    url = talk["slides"].start_with?("http") ? talk["slides"] : "#{BASE_URL}#{talk["slides"]}"

    data["attachment"] << {
      "type" => "Document",
      "url" => talk["slides"],
      "name" => "Prezentacja",
    }
  end

  File.write(path, JSON.pretty_generate(data.compact))
end

def generate_event(event)
  filename = event["date"]
  path = "source/events/#{filename}.json"

  time = Time.parse("#{event["date"]} 18:00")
  time_with_timezone = time.in_time_zone("Europe/Warsaw")

  data = {
    "@context" => "https://www.w3.org/ns/activitystreams",
    "id" => "#{BASE_URL}/events/#{filename}.json",
    "type" => "Event",
    "name" => "TRUG ##{event["number"]}",
    "startTime" => time_with_timezone.iso8601,
    "talks" => [],
    "attributedTo" => "#{BASE_URL}/actors/trug.json"
  }

  data["talks"] =
    (event["talks"] || []).map do |talk|
      filename = "#{event["date"]}-#{(event["talks"].index(talk) + 1).to_s.rjust(3, "0")}"
      {
        "type" => "Note",
        "object" => "#{BASE_URL}/talks/#{filename}.json"
      }
    end

  File.write(path, JSON.pretty_generate(data))
end

def generate_outbox_page(event, next_event)
  filename = event["date"]
  path = "source/outbox/#{filename}.json"

  data = {
    "@context" => "https://www.w3.org/ns/activitystreams",
    "id" => "https://trug.wawer.it/outbox/#{event["date"]}.json",
    "type" => "OrderedCollectionPage",
    "partOf" => "https://trug.wawer.it/outbox",
    "next" => nil,
    "items" => [
      {
        "type" => "Create",
        "object" => "https://trug.wawer.it/events/#{event["date"]}.json"
      }
    ]
  }

  data["next"] = "#{BASE_URL}/outbox/#{next_event["date"]}.json" if next_event

  File.write(path, JSON.pretty_generate(data.compact))
end

namespace :activity_pub do
  task :generate do
    meetups = YAML.load_file("data/meetups.yml")["events"].reverse
    meetups.each_with_index do |meetup|
      next_meetup = meetups[meetups.index(meetup) + 1]
      (meetup["talks"] || {}).each do |talk|
        generate_actor(talk)
        generate_talk(talk, meetup)
      end
      generate_event(meetup)
      generate_outbox_page(meetup, next_meetup)
    end
  end
end
