require 'graphql-docs'

task default: %w[test]

task :test do
    filename = "./schema.graphql"
    output_dir = "./output/"
    landing_pages = Hash.new
    landing_pages[:index] = './docs/index.md'
    GraphQLDocs.build(filename: filename, output_dir: output_dir, landing_pages: landing_pages)
end