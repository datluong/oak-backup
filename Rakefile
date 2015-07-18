require 'dotenv'
require 'aws-sdk'

Dotenv.load

namespace :s3 do
  desc "Upload a file to S3"
  task :s3_upload, [:file_name] do |t, args|
    raise "File not found" unless File.exist? args.file_name
    bucket_name = ENV["AWS_BUCKET"]
    s3 = Aws::S3::Resource.new()
    obj = s3.bucket(bucket_name).object(File.basename(args.file_name))
    puts "Uploading #{args.file_name} to bucket #{bucket_name}.."
    result = obj.upload_file(args.file_name)
    puts "Upload: #{result}"
  end
end
