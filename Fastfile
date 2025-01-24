# frozen_string_literal: true

def root_path
  File.expand_path('..', Dir.pwd)
end

def import_root_file(path)
  import(File.join(root_path, 'fastlane', path))
end

# Import all required files
import_root_file 'helpers.rb'
import_root_file 'configs.rb'
import_root_file 'common_lanes.rb'
import_root_file 'android_lanes.rb'
import_root_file 'ios_lanes.rb'
