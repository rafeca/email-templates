require 'premailer'
require 'pony'

def compile_html(file)
  premailer = Premailer.new(file, {
    :warn_level => Premailer::Warnings::RISKY,
    :base_url => 'http://rafeca.github.com/email-templates/'
  })
  out = premailer.to_inline_css

  # Output any CSS warnings
  premailer.warnings.each do |w|
    puts "#{w[:message]} (#{w[:level]}) may not render properly in #{w[:clients]}"
  end
  out
end


task :build, [:file] do |t, args|
  file = args.file ? args.file : "index"
  fout = File.open("output_#{file}.html", "w")
  fout.puts compile_html(file + ".html")
  fout.close
end

task :sendmail, [:email_recipient, :file] do |t, args|
  if !args.email_recipient
    raise 'usage: rake "sendmail[youemail@example.com]"'
  end

  file = args.file ? args.file : "index"
  body = compile_html(file + ".html")

  Pony.mail(
    :to => args.email_recipient,
    :from => 'me@example.com',
    :subject => 'giffgaff email template',
    :html_body => body
  )
end