markdown_files = (git.modified_files + git.added_files).select do |line|
  line.end_with?(".md")
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
