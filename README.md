<!--
instructions followed: https://feb-dain.github.io/how-to-make-my-github-blog/
-->

# My Blog Journal Page

A repository containing the presentation notes and study log on data science projects at NHS England

## How to run Locally

1. Install [Ruby](https://rubyinstaller.org/downloads/), `Jekyll` and `Bundle` on your system:
    - Open a terminal window
    - Use the command `ruby -v` in your terminal to check which version of the Ruby has been installed
    - Use the command `gem install jekyll bundler` for installation
    - After the installation is done, you can use Jekyll to create and manage static websites, and Bundler to manage dependencies for your Ruby projects.
2. Navigate to a directory named for the gitpage repo: `cd chaeyoonyunakim.github.io`
3. Set a new JeKyll site: `chaeyoonyunakim.github.io>jekyll new ./`
4. In the root directory of your Jekyll site, start building your site: `bundle exec jekyll serve --trace`
    - Jekyll will start building your site and will display output in the terminal. Once the build is complete, it will mention that the development server is running, usually on http://127.0.0.1:4000/.
    - Open a web browser and go to the address shown in the terminal (http://127.0.0.1:4000/). You should be able to see your Jekyll site as it would appear on the web.
5. Clone the repo
    - `git clone https://github.com/chaeyoonyunakim/chaeyoonyunakim.github.io.git`
6. Fine a theme and place the unziped folders and files into your directory: https://jekyllthemes.io/free
7. Apply the theme on your site: bundle exec jekyll serve --trace
8. Make changes to your code using your IDE -save and refresh the page to see the changes locally.
    - `git add --all`
    - `git commit -m "initial commit"`
    - `git push -u origin main`
