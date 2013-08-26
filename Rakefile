require "rake"

desc "Generate an HTML file of the book"
task :gen do
  `cat html/minitestbook-header.html > html/minitestbook.html`
  `redcarpet --parse-fenced_code_blocks --smarty title.md >> html/minitestbook.html`
  `echo '<div class="page_break"></div>' >> html/minitestbook.html`
  `redcarpet --parse-fenced_code_blocks --smarty toc.md >> html/minitestbook.html`
  `echo '<div class="page_break"></div>' >> html/minitestbook.html`
  `redcarpet --parse-fenced_code_blocks --smarty preface.md >> html/minitestbook.html`
  `echo '<div class="page_break"></div>' >> html/minitestbook.html`
  `redcarpet --parse-fenced_code_blocks --smarty prerequisites.md >> html/minitestbook.html`
  `echo '<div class="page_break"></div>' >> html/minitestbook.html`
  `redcarpet --parse-fenced_code_blocks --smarty introduction.md >> html/minitestbook.html`
  `echo '<div class="page_break"></div>' >> html/minitestbook.html`
  `redcarpet --parse-fenced_code_blocks --smarty part1.md >> html/minitestbook.html`
  `echo '<div class="page_break"></div>' >> html/minitestbook.html`
  `redcarpet --parse-fenced_code_blocks --smarty ch1-write-a-test.md >> html/minitestbook.html`
  `echo '<div class="page_break"></div>' >> html/minitestbook.html`
  `redcarpet --parse-fenced_code_blocks --smarty ch2-tdd-red.md >> html/minitestbook.html`
  `echo '<div class="page_break"></div>' >> html/minitestbook.html`
  `redcarpet --parse-fenced_code_blocks --smarty ch3-tdd-green.md >> html/minitestbook.html`
  `echo '<div class="page_break"></div>' >> html/minitestbook.html`
  `redcarpet --parse-fenced_code_blocks --smarty ch4-tdd-right.md >> html/minitestbook.html`
  `echo '<div class="page_break"></div>' >> html/minitestbook.html`
  `redcarpet --parse-fenced_code_blocks --smarty ch5-spec-dsl.md >> html/minitestbook.html`
  `redcarpet --parse-fenced_code_blocks --smarty ch5-spec-dsl.md >> html/minitestbook.html`
  `echo '<div class="page_break"></div>' >> html/minitestbook.html`
  `redcarpet --parse-fenced_code_blocks --smarty appendix/assertions.md >> html/minitestbook.html`
  `echo '<div class="page_break"></div>' >> html/minitestbook.html`
  `redcarpet --parse-fenced_code_blocks --smarty appendix/api-changes.md >> html/minitestbook.html`
  `echo '<div class="page_break"></div>' >> html/minitestbook.html`
  `echo '<small><pre>' >> html/minitestbook.html`
  `cat LICENSE >> html/minitestbook.html`
  `echo '</pre></small>' >> html/minitestbook.html`
  `cat html/minitestbook-footer.html >> html/minitestbook.html`
  `open html/minitestbook.html`
end
