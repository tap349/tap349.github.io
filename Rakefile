namespace :post do
  desc 'Create new blog post'
  task :create, [:title, :access] do |_t, args|
    args.with_defaults access: :public

    title = args.title.gsub ' ', '-'
    date = Time.new.strftime '%Y-%m-%d'
    filename = "_posts/#{date}-#{title}.md"

    File.open filename, 'w' do |file|
      file.write "
        ---
        layout: post
        title: #{args.title}
        date: #{Time.new}
        access: #{args.access}
        categories: []
        ---
      ".gsub(/^\s+/, '')
    end

    `mvim #{filename}`
  end
end
