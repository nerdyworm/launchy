require 's3'
require 'digest/md5'
require 'mime/types'

YUI_COMPRESSOR_JAR = "~/lib/yuicompressor.jar"

AWS_ACCESS_KEY_ID = ENV['AWS_ACCESS_KEY']
AWS_SECRET_ACCESS_KEY = ENV['AWS_SECRET_KEY']
AWS_BUCKET = "static.whotouse.co"



task :deploy => [:prepare, :static, :css, :img, :s3upload, :s3ify_html,
  :set_meta, :www_upload, :clean] do

end

desc "Prepare for deploy"
task :prepare do
  sh "mkdir -p tmp/public/"
  sh "mkdir -p tmp/assets/img/"
  sh "mkdir -p tmp/assets/css/"
end

desc "Move all static files to tmp pubblic"
task :static do
  sh "cp public/*.html tmp/public/"
  sh "cp public/*.txt  tmp/public/"
  sh "cp public/*.ico  tmp/public/"
end

desc 'Pack stylesheets to assets folder"'
task :css do
  sh "java -jar #{YUI_COMPRESSOR_JAR} public/css/style.css > tmp/assets/css/style.min.css"
end

desc "Copy image assets folder"
task :img do
  sh "cp public/img/* tmp/assets/img/"
end

desc "Make all html assets point to s3"
task :s3ify_html do
  Dir.glob("tmp/public/*.html").each do |file|
    if File.file?(file)
      text = File.read(file)

      text = text.gsub(/href="css\//, "href=\"http://static.whotouse.co/css/")
      text = text.gsub(/\.css"/, ".min.css\"")
      text = text.gsub(/<img src="img\//, "<img src=\"http://static.whotouse.co/img/")

      File.open(file, "w") {|file| file.puts text }
    end
  end
end

desc "Set meta content"
task :set_meta do
  Dir.glob("tmp/public/*.html").each do |file|
    if File.file?(file)
      text = File.read(file)

      text = text.gsub('<meta name="description" content="">',
                       '<meta name="description" content="Who to use"/>')
      text = text.gsub('<meta name="author" content="">',
                       '<meta name="author" content="Who to use - Will Gregg, Benjamin Rhodes"/>')

      File.open(file, "w") {|file| file.puts text }
    end
  end
end


desc "Upload all changed assets in assts/**/* to S3"
task :s3upload do
  puts "== Uploading assets to S3/Cloudfront"
  service = S3::Service.new(
    :access_key_id => AWS_ACCESS_KEY_ID,
    :secret_access_key => AWS_SECRET_ACCESS_KEY)
    bucket = service.buckets.find(AWS_BUCKET)

    STDOUT.sync = true

    Dir.glob("tmp/assets/**/*").each do |file|
      if File.file?(file)

        remote_file = file.gsub("tmp/assets/", "")

        begin
          obj = bucket.objects.find_first(remote_file)
        rescue
          obj = nil
        end

        if !obj || (obj.etag != Digest::MD5.hexdigest(File.read(file)))
          print "U"

          obj = bucket.objects.build(remote_file)
          obj.content = open(file)
          obj.content_type = MIME::Types.type_for(file).first.to_s
          obj.save
        else
          print "."
        end
      end
    end
    STDOUT.sync = false

    puts
    puts "== Done syncing assets"
end


desc "Upload to the www server"
task :www_upload do
  sh "cd tmp && tar -zcvf public.tar.gz public/*"
  sh 'cat tmp/public.tar.gz | ssh whotouse.co "cd /data/apps/whotouse.co ; tar zxvf - "'
end

desc "Clean up"
task :clean do
  sh "rm -rf tmp/"
end
