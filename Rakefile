require 'premailer'
require 'pony'
require 'fileutils'
require 'yaml'

BASE_URL = 'http://rafeca.github.com/email-templates/'

def compile_html(file, fileout)
  premailer = Premailer.new(file, {
    :warn_level => Premailer::Warnings::RISKY,
    :base_url => BASE_URL
  })
  out = premailer.to_inline_css

  # Output any CSS warnings
  premailer.warnings.each do |w|
    puts "#{w[:message]} (#{w[:level]}) may not render properly in #{w[:clients]}"
  end

  File.open(fileout, "w") {|file| file.puts out}
  out
end

def embed_responsive_css(file, fileout)

  html = File.read(file)

  # custom transforms
  if File.file?('replacements.yml')
    transforms = YAML.load_file('replacements.yml')
    transforms.each do |from, to|
      html.gsub!(from, to)
    end
  end

  # transform absolute links
  html.gsub!('src="/', 'src="')
  html.gsub!('href="/', 'href="')
  html.gsub!('url(', 'url(' + BASE_URL)

  # embed responsive stylesheet into html
  css = '<style data-premailer="ignore" type="text/css">' + "\n"
  css += File.read('css/responsive.css')
  css += "\n" + '</style>'
  css.gsub!("url(../", "url(" + BASE_URL)
  html.gsub!('<link rel="stylesheet" type="text/css" href="css/responsive.css">', css)

  File.open(fileout, "w") {|file| file.puts html}
end

def build_email(file)
  FileUtils::mkdir_p "dist/"
  file = (file ? file : "index") + ".html"
  filetmp = "tmp_" + File.basename(file)
  fileout = "dist/" + File.basename(file)

  embed_responsive_css(file, filetmp)
  out = compile_html(filetmp, fileout)
  File.delete(filetmp)

  out
end

# Begin actual tasks

task :build, [:file] do |t, args|
  build_email(args.file)
end

task :sendmail, [:email_recipient, :file] do |t, args|
  if !args.email_recipient
    raise 'usage: rake "sendmail[youemail@example.com]"'
  end

  body = build_email(args.file)

  Pony.mail(
    :to => args.email_recipient,
    :subject => 'giffgaff email template',
    :html_body => body
  )
end

task :server do |t|
  exec('ruby -run -e httpd . -p 5000')
end
