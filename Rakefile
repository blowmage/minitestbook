require "rake"

desc "Generate an HTML file of the book"
task :gen do
  `cat html/header_new.html > html/minitestbook.html`
  `redcarpet --parse-fenced_code_blocks --smarty title.md >> html/minitestbook.html`
  `echo '<div class="page_break"></div>' >> html/minitestbook.html`
  `redcarpet --parse-fenced_code_blocks --smarty toc.md >> html/minitestbook.html`
  `echo '<div class="page_break"></div>' >> html/minitestbook.html`
  `redcarpet --parse-fenced_code_blocks --smarty preface.md >> html/minitestbook.html`
  `echo '<div class="page_break"></div>' >> html/minitestbook.html`
  `redcarpet --parse-fenced_code_blocks --smarty introduction.md >> html/minitestbook.html`
  `echo '<div if="part1" class="page_break"></div>' >> html/minitestbook.html`
  `redcarpet --parse-fenced_code_blocks --smarty part1.md >> html/minitestbook.html`
  `echo '<div id="ch1-write-a-test" class="page_break"></div>' >> html/minitestbook.html`
  `redcarpet --parse-fenced_code_blocks --smarty ch1-write-a-test.md >> html/minitestbook.html`
  `echo '<div id="ch2-tdd-red-green" class="page_break"></div>' >> html/minitestbook.html`
  `redcarpet --parse-fenced_code_blocks --smarty ch2-tdd-red-green.md >> html/minitestbook.html`
  `echo '<div id="ch3-minibot-rules" class="page_break"></div>' >> html/minitestbook.html`
  `redcarpet --parse-fenced_code_blocks --smarty ch3-minibot-rules.md >> html/minitestbook.html`
  `echo '<div id="ch4-minibot-responses" class="page_break"></div>' >> html/minitestbook.html`
  `redcarpet --parse-fenced_code_blocks --smarty ch4-minibot-responses.md >> html/minitestbook.html`
  `echo '<div id="ch5-tdd-refactor" class="page_break"></div>' >> html/minitestbook.html`
  `redcarpet --parse-fenced_code_blocks --smarty ch5-tdd-refactor.md >> html/minitestbook.html`
  `echo '<div id="ch6-tdd-refactor-redux" class="page_break"></div>' >> html/minitestbook.html`
  `redcarpet --parse-fenced_code_blocks --smarty ch6-tdd-refactor-redux.md >> html/minitestbook.html`
  `echo '<div id="ch7-spec-dsl" class="page_break"></div>' >> html/minitestbook.html`
  `redcarpet --parse-fenced_code_blocks --smarty ch7-spec-dsl.md >> html/minitestbook.html`
  `echo '<div class="page_break"></div>' >> html/minitestbook.html`
  # `redcarpet --parse-fenced_code_blocks --smarty appendix/assertions.md >> html/minitestbook.html`
  # `echo '<div class="page_break"></div>' >> html/minitestbook.html`
  # `redcarpet --parse-fenced_code_blocks --smarty appendix/api-changes.md >> html/minitestbook.html`
  # `echo '<div class="page_break"></div>' >> html/minitestbook.html`
  `echo '<small><pre>' >> html/minitestbook.html`
  `cat LICENSE >> html/minitestbook.html`
  `echo '</pre></small>' >> html/minitestbook.html`
  `cat html/minitestbook-footer.html >> html/minitestbook.html`
  `open html/minitestbook.html`
end










