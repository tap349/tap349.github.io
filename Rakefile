namespace :post do
  desc 'Create new blog post'
  task :create, [:title, :access] do |_t, args|
    args.with_defaults access: :public

    title = args.title.gsub(' ', '-').downcase
    date = Time.new.strftime '%Y-%m-%d'
    filename = "_posts/#{date}-#{title}.md"

    File.open filename, 'w' do |file|
      file.write <<~TEMPLATE
        ---
        layout: post
        title: #{args.title}
        date: #{Time.new}
        access: #{args.access}
        categories: []
        ---

        <!-- more -->
      TEMPLATE
    end
  end
end
