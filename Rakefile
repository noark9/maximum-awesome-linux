require 'tmpdir'

def step(description)
  description = "-- #{description} "
  description = description.ljust(80, '-')
  puts
  puts "\e[32m#{description}\e[0m"
end

def step(description)
  description = "-- #{description} "
  description = description.ljust(80, '-')
  puts
  puts "\e[32m#{description}\e[0m"
end

def install_github_bundle(user, package)
  unless File.exist? File.expand_path("~/.vim/bundle/#{package}")
    sh "git clone https://github.com/#{user}/#{package} ~/.vim/bundle/#{package}"
  end
end

def link_file(original_filename, symlink_filename)
  original_path = File.expand_path(original_filename)
  symlink_path = File.expand_path(symlink_filename)
  if File.exists?(symlink_path) || File.symlink?(symlink_path)
    if File.symlink?(symlink_path)
      symlink_points_to_path = File.readlink(symlink_path)
      return if symlink_points_to_path == original_path
      # Symlinks can't just be moved like regular files. Recreate old one, and
      # remove it.
      ln_s symlink_points_to_path, get_backup_path(symlink_path), :verbose => true
      rm symlink_path
    else
      # Never move user's files without creating backups first
      mv symlink_path, get_backup_path(symlink_path), :verbose => true
    end
  end
  ln_s original_path, symlink_path, :verbose => true
end

namespace :install do

  desc 'dnf Update'
  task :update do
    step 'dnf update'
    sh 'sudo dnf -y update'
  end

  desc 'Install Vim'
  task :vim do
    step 'vim'
    sh 'sudo dnf install -y vim'
  end

  desc 'Install tmux'
  task :tmux do
    step 'tmux'
    sh 'sudo dnf install -y tmux'
  end

  desc 'Install ctags'
  task :ctags do
    step 'ctags'
    sh 'sudo dnf install -y ctags'
  end

  # https://github.com/ggreer/the_silver_searcher
  desc 'Install The Silver Searcher'
  task :the_silver_searcher do
    step 'the_silver_searcher'
    sh 'sudo dnf install -y the_silver_searcher'
  end
  
  desc 'Install Vundle'
  task :vundle do
    step 'vundle'
    install_github_bundle 'VundleVim','Vundle.vim'
    sh '/usr/bin/vim -c "PluginInstall!" -c "q" -c "q"'
  end

  # instructions from http://www.webupd8.org/2011/04/solarized-must-have-color-paletter-for.html
  desc 'Install Solarized and fix ls'
  task :solarized, :arg1 do |t, args|
    args[:arg1] = "dark" unless ["dark", "light"].include? args[:arg1]
    color = ["dark", "light"].include?(args[:arg1]) ? args[:arg1] : "dark"

    step 'solarized'
    sh 'git clone https://github.com/sigurdga/gnome-terminal-colors-solarized.git' unless File.exist? 'gnome-terminal-colors-solarized'
    Dir.chdir 'gnome-terminal-colors-solarized' do
      sh "./solarize #{color}"
    end

    step 'fix ls-colors'
    Dir.chdir do
      sh "wget --no-check-certificate https://raw.github.com/seebi/dircolors-solarized/master/dircolors.ansi-#{color}"
      sh "mv dircolors.ansi-#{color} .dircolors"
      sh 'eval `dircolors .dircolors`'
    end
  end
end

def filemap(map)
  map.inject({}) do |result, (key, value)|
    result[File.expand_path(key)] = File.expand_path(value)
    result
  end.freeze
end

COPIED_FILES = filemap(
  'vimrc.local'         => '~/.vimrc.local',
  'vimrc.bundles.local' => '~/.vimrc.bundles.local',
  'tmux.conf.local'     => '~/.tmux.conf.local'
)

LINKED_FILES = filemap(
  'vim'           => '~/.vim',
  'tmux.conf'     => '~/.tmux.conf',
  'vimrc'         => '~/.vimrc',
  'vimrc.bundles' => '~/.vimrc.bundles'
)

desc 'Install these config files.'
task :default do
  Rake::Task['install:update'].invoke
  Rake::Task['install:vim'].invoke
  Rake::Task['install:tmux'].invoke
  Rake::Task['install:ctags'].invoke
  Rake::Task['install:the_silver_searcher'].invoke

  step 'symlink'

  LINKED_FILES.each do |orig, link|
    link_file orig, link
  end

  COPIED_FILES.each do |orig, copy|
    cp orig, copy, :verbose => true unless File.exist?(copy)
  end
  
  Rake::Task['install:vundle'].invoke

  step 'solarized dark or light'
  puts
  puts " You're almost done! Inside of the maximum-awesome-linux directory, do: "
  puts "   rake install:solarized['dark'] "
  puts "     or                           "
  puts "   rake install:solarized['light']"

  puts " You may need to close your terminal and re-open it for it to take effect."
end
