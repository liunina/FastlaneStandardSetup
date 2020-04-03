# This file contains the fastlane.tools configuration
# You can find the documentation at https://docs.fastlane.tools
#
# For a list of all available actions, check out
#
#     https://docs.fastlane.tools/actions
#
# For a list of all available plugins, check out
#
#     https://docs.fastlane.tools/plugins/available-plugins
#

# Uncomment the line if you want fastlane to automatically update itself
# update_fastlane

default_platform(:ios)

platform :ios do
  #  fastlane release_pod project:'框架名' version:'版本'
  #  fastlane release_pod repo:NLSpecs project:'框架名' version:'版本'
  desc "Release new private pod version"
  lane :release_pod do |options|
    target_repo    = options[:repo]
    target_project = options[:project]
    target_version = options[:version]
    target_desc    = options[:desc]
    target_verbose    = options[:verbose]
    spec_path = "#{target_project}.podspec"
    
    if target_project.nil? || target_project.empty? || target_version.nil? || target_version.empty?
      UI.message("❌ Project name and version number are required parameters")
      exit
    end

    UI.message("👉 Start release lib #{target_project} new version #{target_version}")
    
    cocoapods(use_bundle_exec: false)
    # 1.  git pull
    git_pull 
    # 2.  确认是 master 分支
    ensure_git_branch
    # 3.  修改 spec 为即将发布的版本
    version_bump_podspec(path: spec_path, version_number: target_version)
    # 4.  pod install
    cocoapods(
      clean: true,
      repo_update: true,
      podfile: "./Example/Podfile"
    )
    # 5.  提交代码到远程仓库
    git_add(path: '.')

    begin
      if target_desc.nil? || target_desc.empty?
        git_commit(path: '.', message: "release #{target_version}")
      else
        git_commit(path: '.', message: target_desc)
      end
      rescue
        error_message = "#{$!}"
        UI.message("⚠️ commit error:#{error_message}")
        unless error_message.include?("nothing to commit, working directory clean")
        exit
        end
        UI.message("The local code has committed, skip the commit step!")
    end
      
    push_to_git_remote
    # 6.  检查对于 tag 是否已经存在
    # 6.1  验证当前tag是否存在，如果说存在的话，干掉，进行下一步，创建一个tag，如果说不存在的话，直接创建tag
    if git_tag_exists(tag: target_version)
      UI.message("发现 tag:#{target_version} 存在，即将执行删除动作 🚀")
      # 6.2  下面的两种写法都可以
      remove_git_tag(tag:target_version,isRL:true,isRR:true)
      #remove_tag(tag:tagName)
    end
    # 7.  添加 tag
    add_git_tag(tag: target_version)
    
    # 8.  提交 tag
    push_git_tags
    
    t_verbose = true
    target_sources = ["https://github.com/CocoaPods/Specs.git", "https://github.com/liunina/NLSpecs.git", "http://nas.iliunian.com:82/GiHoo/GiHooSpecs.git"];
    # 9.  本地验证 spec 文件
    if target_verbose.nil? || target_verbose.empty?
      t_verbose = false
    else
      t_verbose = true
    end

    pod_lib_lint(allow_warnings: true, verbose: t_verbose, sources: target_sources)

    # 10.检查是否传了 repo 参数
    if target_repo
      # 10.1  pod repo push 'target_repo' 'spec_path'
      pod_push(path: spec_path, repo: target_repo, allow_warnings: true, sources: target_sources)
      UI.message("Release lib #{target_project} new version #{target_version} to repo #{target_repo} Successfully! 🎉 ")
    else
      # 10.2  pod trunk push 'spec_path'
      pod_push(path: spec_path, allow_warnings: true, sources: ["https://github.com/CocoaPods/Specs.git"])
      UI.message("Release lib #{target_project} new version #{target_version} to CocoaPods/Specs Successfully! 🎉 ")
    end
  end
end
