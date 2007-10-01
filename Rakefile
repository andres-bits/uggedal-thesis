SRC_DIR = File.expand_path(pwd + '/src')
BASE_FILE = 'base'
PROJECT_FILE = File.basename(pwd)
TMP_FILES = %w(aux bbl blg log lot lof toc)
GEN_FILES = %w(dvi ps pdf)

task :default => 'compile:dvi'

desc 'Compiles source files with latex and bibtex'
task :compile do
  cd(SRC_DIR) do
    system "latex #{BASE_FILE}"
    if FileList['*.bib'].size > 0
      system "bibtex #{BASE_FILE}"
      system "latex #{BASE_FILE}"
    end
    system "latex #{BASE_FILE}"
  end
end

namespace :compile do

  desc 'Compiles source files into dvi format'
  task :dvi => :compile do
    mv_file(:dvi)
  end

  desc 'Compiles source files into ps format'
  task :ps => :compile do
    cd(SRC_DIR) do
      system "dvips #{BASE_FILE}"
    end
    mv_file(:ps)
  end

  desc 'Compiles source files into pdf format'
  task :pdf => :compile do
    cd(SRC_DIR) do
      system "pdflatex #{BASE_FILE}"
    end
    mv_file(:pdf)
  end
end

desc 'View compiled file'
task :view => 'view:dvi'

namespace :view do

  desc 'View compiled file in dvi format'
  task :dvi => 'compile:dvi'do
    view_file(:dvi)
  end

  desc 'View compiled file in ps format'
  task :ps => 'compile:ps' do
    view_file(:ps)
  end

  desc 'View compiled file in pdf format'
  task :pdf => 'compile:pdf' do
    view_file(:pdf)
  end
end

desc 'Spell check source files'
task :spell do
  # TODO: use ispell -t
end

desc 'Generate a statistical report'
task :report do
  print_stats_for_files(FileList["#{SRC_DIR}/*.tex"])
  # TODO: word count: per chapter, with and without appendix
end

desc 'Cleans up temporary files in source directory'
task :clean do
  cd(SRC_DIR) do
    auxilary_files = TMP_FILES + GEN_FILES
    auxilary_files.each do |tmp_file|
      rm_f FileList["*.#{tmp_file}"]
    end
  end
end

desc 'Cleans up explicitly created files in base directory'
task :clobber => :clean do
  GEN_FILES.each do |gen_file|
    rm_f FileList["#{PROJECT_FILE}.#{gen_file}"]
  end
end

private

  # Moves a base file in the source directory of the given format to the base
  # directory, renaming it to use the project file name.
  def mv_file(format)
    mv(File.join(SRC_DIR, "#{BASE_FILE}.#{format.to_s}"),
       File.join(pwd, "/#{PROJECT_FILE}.#{format.to_s}"))
  end

  # Opens up a compiled file of the given format in an appropriate viewer.
  def view_file(format)
    viewers = case format
              when :dvi
                %w(dviprog xdvi)
              when :ps
                %w(psprog evince acroread)
              when :pdf
                %w(kpdf evince acroread)
              end
    viewers.each do |viewer|
      if system "which #{viewer}"
        system "#{viewer} #{PROJECT_FILE}.#{format.to_s}"
        return
      end
    end
  end

  # Prints a statistical table for the given files.
  def print_stats_for_files(files)
    print_stat_splitter
    print_stat_header
    print_stat_splitter
    total_wc, textual_wc = 0, 0
    files.each do |file|
      stats = {}
      stats[:full_wc], stats[:textual_wc] = calculate_word_count(file)
      stats[:textual_percentage] = to_percent(stats[:full_wc],
                                              stats[:textual_wc])
      print_stat_line(File.basename(file), stats)
      total_wc += stats[:full_wc]
      textual_wc += stats[:textual_wc]
    end
    print_stat_splitter
    print_stat_line('Total',
                    { :full_wc => total_wc,
                      :textual_wc => textual_wc,
                      :textual_percentage => to_percent(total_wc,
                                                        textual_wc) })
    print_stat_splitter
  end

  def print_stat_line(name, stats)
    puts "| #{name.ljust(30)}" +
         "| #{stats[:full_wc].to_s.rjust(8)}" +
         "| #{stats[:textual_wc].to_s.rjust(8)}" +
         "| #{stats[:textual_percentage].to_s.rjust(8)} |"
  end

  def print_stat_header
    print_stat_line('Filename', { :full_wc => 'Total',
                                  :textual_wc => 'Textual',
                                  :latex_wc => 'LaTeX',
                                  :textual_percentage => 'Text %' })
  end

  def print_stat_splitter
    puts '+' + '-'*22 + ('-'*9 + '+') * 3 + '-'*10 + '+'
  end
  
  # Calculates full and textual word count for a given latex file.
  def calculate_word_count(file)
    full_wc, textual_wc = 0, 0
    File.open(file) do |f|
      f.each do |l|
        full_wc += l.split.size
        unless l.lstrip =~ /^%/         # don't count comments
          l.gsub!(/\\\w+/, '')          # strip latex commands
          l.gsub!(/\[[\w,]+\]/, '')     # strip latex command options
          l.gsub!(/\{[\w,.:_-]+\}/, '') # strip latex command parameters
          l.gsub!(/&/, '')              # strip latex column seperator
          l.gsub!(/\\\\/, '')           # strip latex implicit line break
          textual_wc += l.split.size
        end
      end
    end
    [full_wc, textual_wc]
  end

  def to_percent(major, minor)
    unless major == 0
      (100 * minor / major).round
    else
      0
    end
  end