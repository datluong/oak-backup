require 'dotenv'
require 'aws-sdk'

Dotenv.load

today_postfix = Time.now.utc.strftime "%m_%d_%Y"
backup_filename = "mongo_backup_#{today_postfix}.tar.gz"

def prettify_byte(s)
  str = s.to_s.strip
  return str if str.size < 6
  regex = /(\d\d\d)$/
  comps = []
  while str =~ regex
    comps << regex.match(str).to_s
    str.gsub!( regex, "" )
  end
  comps << str if str.size > 0
  comps.reverse.join(",")
end

namespace :backup do

  task :clean do
    puts "Task[clean] Cleaning.."
    rm_rf "dump", :verbose => true
    rm_rf "tmp", :verbose => true
  end

  task :create => [:clean] do
    dir = "tmp/#{today_postfix}/"
    mkdir_p dir
    host = ENV['MONGODB_HOSTNAME'] || "localhost"
    port = ENV['MONGODB_PORT'] || 27017
    db   = ENV['MONGODB_DATABASE']
    user = ENV['MONGODB_USERNAME']
    password = ENV['MONGODB_PASSWORD']
    cmd = "mongodump -h #{host}:#{port} -d #{db} -u #{user} -p #{password}"
    sh cmd
    mv "dump", dir
  end

  # Compress dump files
  task :compress => [:create] do
    gz_name = "tmp/#{backup_filename}"
    dir     = "tmp/#{today_postfix}/"
    cmd = "tar -cvzf #{gz_name} #{dir}"
    sh cmd
  end

  task :upload => [:compress] do
    gz_name = "tmp/#{backup_filename}"
    Rake::Task["s3:upload"].invoke(gz_name)
    Rake::Task["backup:clean"].reenable
    Rake::Task["backup:clean"].invoke
  end

end

namespace :s3 do
  desc "Report backup creation status"
  task :report do
    s3 = Aws::S3::Client.new
    bucket_name = ENV["AWS_BUCKET"]
    obj = s3.list_objects(:bucket => bucket_name).contents.select { |t| t.key == backup_filename }.first
    if obj.nil?
      puts "Not found on AWS bucket #{bucket_name}"
    else
      puts "Backup created successful (#{obj.key}) Size: #{prettify_byte(obj.size)} byte(s)"
    end
  end

  desc "List file in bucket"
  task :list do
    s3 = Aws::S3::Client.new
    bucket_name = ENV["AWS_BUCKET"]
    s3.list_objects(:bucket => bucket_name).contents.each do |t|
      puts "#{t.key}, size #{t.size}"
    end
  end

  desc "Upload a file to S3"
  task :upload, [:file_name] do |t, args|
    raise "File not found" unless File.exist? args.file_name
    bucket_name = ENV["AWS_BUCKET"]
    s3 = Aws::S3::Resource.new()
    obj = s3.bucket(bucket_name).object(File.basename(args.file_name))
    puts "Uploading #{args.file_name} to bucket #{bucket_name}.."
    result = obj.upload_file(args.file_name)
    puts "Uploaded: #{result}"
    result
  end
end
