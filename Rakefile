require 'graphql-docs'

task default: %w[build]

task :build do
    puts "building docs"
    filename = "./schema.graphql"
    output_dir = "./docs/"
    landing_pages = Hash.new
    landing_pages[:index] = './src/index.md'
    base_url = "/graphql-api-docs"
    GraphQLDocs.build(filename: filename, output_dir: output_dir, landing_pages: landing_pages, base_url: base_url)
end

task :publish do
    sh "git add ."
    sh "git commit -m 'Update docs'"
    sh "git push"
end

task update: %w[publish build]