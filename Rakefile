task :default do
	puts "Running CI tasks..."

	sh("JEKYLL_ENV=production bundle exec jekyll build")
	HTMLProofer.check_directory(
    "./_site",
    url_ignore: [/linkedin.com|php-fig.org|bower.io|bost.ocks.org|elementary.io/] 
  ).run
	puts "Jekyll successfully built"
end