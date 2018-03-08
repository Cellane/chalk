markdown_files = (git.modified_files + git.added_files).select do |line|
  line.end_with?(".md")
end

published_posts = git.renamed_files.select do |entry|
  entry[:before].start_with?("_drafts") && entry[:after].start_with?("_posts")
end

if github.pr_title =~ /\[#\d+\]/ && published_posts.length == 0
  warn 'It looks like you\'re trying to merge an article but probably forgot to publish it?'
end

ignored_words = []

File.open('.spelling').each_line do |line|
  line.chomp!
  next if line.empty? || line.start_with?('#')
  ignored_words.push(line)
end

prose.ignored_words = ignored_words
prose.ignore_numbers = true
prose.ignore_acronyms = true
prose.lint_files markdown_files
prose.check_spelling markdown_files
