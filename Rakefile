require 'graphql-docs'

task default: %w[test]

task :test do
    filename = "./schema.graphql"
    output_dir = "./docs/"
    landing_pages = Hash.new
    landing_pages[:index] = './src/index.md'
    GraphQLDocs.build(filename: filename, output_dir: output_dir, landing_pages: landing_pages)
end