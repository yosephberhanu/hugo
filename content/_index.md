---
# Leave the homepage title empty to use the site title
title: ''
date: 2022-10-24
type: landing

sections:
  - block: about.biography
    id: about
    content:
      title: Biography
      # Choose a user profile to display (a folder name within `content/authors/`)
      username: admin
  - block: skills
    content:
      title: Skills
      text: ''
      # Choose a user to display skills from (a folder name within `content/authors/`)
      username: admin
    design:
      columns: '1'
  - block: experience
    content:
      title: Experience
      # Date format for experience
      #   Refer to https://docs.hugoblox.com/customization/#date-format
      date_format: Jan 2006
      # Experiences.
      #   Add/remove as many `experience` items below as you like.
      #   Required fields are `title`, `company`, and `date_start`.
      #   Leave `date_end` empty if it's your current employer.
      #   Begin multi-line descriptions with YAML's `|2-` multi-line prefix.
      items:
        - title: Graduate Research/Teaching Assistant
          company: Virginia Tech
          company_url: 'https://www.vt.edu'
          company_logo: org-vt
          location: Blacksburg, VA
          date_start: '2023-08-25'
          date_end: ''
          description: |2-
              Responsibilities include:

              * Analysing
              * Modelling
              * Deploying
        - title: Lecturer
          company: Addis Ababa University 
          company_url: 'https://www.aau.edu.et'
          company_logo: org-aau
          location: Addis Ababa, Ethiopia
          date_start: '2014-06-15'
          date_end: '2023-07-30'
          description: Taught electronic engineering and researched semiconductor physics.
    design:
      columns: '2'
  # - block: accomplishments
  #   content:
  #     # Note: `&shy;` is used to add a 'soft' hyphen in a long heading.
  #     title: 'Accomplish&shy;ments'
  #     subtitle:
  #     # Date format: https://docs.hugoblox.com/customization/#date-format
  #     date_format: Jan 2006
  #     # Accomplishments.
  #     #   Add/remove as many `item` blocks below as you like.
  #     #   `title`, `organization`, and `date_start` are the required parameters.
  #     #   Leave other parameters empty if not required.
  #     #   Begin multi-line descriptions with YAML's `|2-` multi-line prefix.
  #     items:
  #       - certificate_url: https://www.coursera.org
  #         date_end: ''
  #         date_start: '2021-01-25'
  #         description: ''
  #         icon: coursera
  #         organization: Coursera
  #         organization_url: https://www.coursera.org
  #         title: Neural Networks and Deep Learning
  #         url: ''
  #       - certificate_url: https://www.edx.org
  #         date_end: ''
  #         date_start: '2021-01-01'
  #         description: Formulated informed blockchain models, hypotheses, and use cases.
  #         icon: edx
  #         organization: edX
  #         organization_url: https://www.edx.org
  #         title: Blockchain Fundamentals
  #         url: https://www.edx.org/professional-certificate/uc-berkeleyx-blockchain-fundamentals
  #       - certificate_url: https://www.datacamp.com
  #         date_end: '2020-12-21'
  #         date_start: '2020-07-01'
  #         description: ''
  #         icon: datacamp
  #         organization: DataCamp
  #         organization_url: https://www.datacamp.com
  #         title: 'Object-Oriented Programming in R'
  #         url: ''
  #   design:
  #     columns: '2'
  - block: collection
    id: posts
    content:
      title: Recent Posts
      subtitle: ''
      text: ''
      # Choose how many pages you would like to display (0 = all pages)
      count: 3
      # Filter on criteria
      filters:
        folders:
          - post
        author: ""
        category: ""
        tag: ""
        exclude_featured: false
        exclude_future: false
        exclude_past: false
        publication_type: ""
      # Choose how many pages you would like to offset by
      offset: 0
      # Page order: descending (desc) or ascending (asc) date.
      order: desc
    design:
      # Choose a layout view
      view: compact
      columns: '2'
  - block: portfolio
    id: projects
    content:
      title: Projects
      filters:
        folders:
          - project
      # Default filter index (e.g. 0 corresponds to the first `filter_button` instance below).
      default_button_index: 0
      # Filter toolbar (optional).
      # Add or remove as many filters (`filter_button` instances) as you like.
      # To show all items, set `tag` to "*".
      # To filter by a specific tag, set `tag` to an existing tag name.
      # To remove the toolbar, delete the entire `filter_button` block.
      buttons:
        - name: All
          tag: '*'
        - name: Deep Learning
          tag: Deep Learning
        - name: Other
          tag: Demo
    design:
      # Choose how many columns the section has. Valid values: '1' or '2'.
      columns: '1'
      view: showcase
      # For Showcase view, flip alternate rows?
      flip_alt_rows: false
  
  # - block: collection
  #   id: featured
  #   content:
  #     title: Featured Publications
  #     filters:
  #       folders:
  #         - publication
  #       featured_only: true
  #   design:
  #     columns: '2'
  #     view: card
  - block: collection
    content:
      title: Publications
      text: |-
        {{% callout note %}}
        Quickly discover relevant content by [filtering publications](./publication/).
        {{% /callout %}}
      filters:
        folders:
          - publication
        exclude_featured: true
    design:
      columns: '2'
      view: citation
  - block: collection
    id: teaching
    content:
      title: Teaching
      text: ''
      filters:
        folders:
          - courses
        exclude_featured: true
    design:
      columns: '2'
      # view: citation
  # - block: collection
  #   id: talks
  #   content:
  #     title: Recent & Upcoming Talks
  #     filters:
  #       folders:
  #         - event
  #   design:
  #     columns: '2'
  #     view: compact
  # - block: tag_cloud
  #   content:
  #     title: Popular Topics
  #   design:
  #     columns: '2'
  - block: contact
    id: contact
    content:
      title: Contact
      subtitle:
      text: |-
        Interested in collaborating or have questions? I'd love to hear from you! Please feel free to reach out through the contact form below.
      # Contact (add or remove contact options as necessary)
      # email: yoseph at vt
      # phone: 888 888 88 88
      # appointment_url: 'https://calendly.com'
      # address:
      #   street: 450 Serra Mall
      #   city: Stanford
      #   region: CA
      #   postcode: '94305'
      #   country: United States
      #   country_code: US
      # directions: Enter Building 1 and take the stairs to Office 200 on Floor 2
      # office_hours:
      #   - 'Monday 10:00 to 13:00'
      #   - 'Wednesday 09:00 to 10:00'
      # Choose a map provider in `params.yaml` to show a map from these coordinates
      # coordinates:
      #   latitude: '37.4275'
      #   longitude: '-122.1697'  
      # contact_links:
      #   - icon: twitter
      #     icon_pack: fab
      #     name: DM Me
      #     link: 'https://twitter.com/yosephberhanu'
      #   - icon: video
      #     icon_pack: fas
      #     name: Zoom Me
      #     link: 'https://zoom.com'
      #   - icon: skype
      #     icon_pack: fab
      #     name: Skype Me
      #     link: 'skype:echo123?call'
      # Automatically link email and phone or display as text?
      autolink: true
      # Email form provider
      form:
        provider: netlify
        formspree:
          id:
        netlify:
          # Enable CAPTCHA challenge to reduce spam?
          captcha: false
    design:
      columns: '2'
  # - block: hero
  #   demo: true # Only display this section in the Hugo Blox Builder demo site
  #   content:
  #     title: Hugo Academic Theme
  #     image:
  #       filename: hero-academic.png
  #     cta:
  #       label: '**Get Started**'
  #       url: https://hugoblox.com/templates/
  #     cta_alt:
  #       label: Ask a question
  #       url: https://discord.gg/z8wNYzb
  #     cta_note:
  #       label: >-
  #         <div style="text-shadow: none;"><a target="_blank" class="github-button" href="https://github.com/HugoBlox/hugo-blox-builder" data-icon="octicon-star" data-size="large" data-show-count="true" aria-label="Star">Star Hugo Blox Builder</a></div><div style="text-shadow: none;"><a target="_blank" class="github-button" href="https://github.com/HugoBlox/theme-academic-cv" data-icon="octicon-star" data-size="large" data-show-count="true" aria-label="Star">Star the Academic template</a></div>
  #     text: |-
  #       **Generated by Hugo Blox Builder - the FREE, Hugo-based open source website builder trusted by 500,000+ sites.**

  #       **Easily build anything with blocks - no-code required!**

  #       From landing pages, second brains, and courses to academic resumés, conferences, and tech blogs.

  #       <!--Custom spacing-->
  #       <div class="mb-3"></div>
  #       <!--GitHub Button JS-->
  #       <script async defer src="/media/https://buttons.github.io/buttons.js"></script>
  #   design:
  #     background:
  #       gradient_end: '#1976d2'
  #       gradient_start: '#004ba0'
  #       text_color_light: true
  - block: markdown
    content:
      title: Advisee tree (Fun fact)
      subtitle: ''
      text: <style>.family-tree ul{padding-top:20px;position:relative;transition:all 0.5s;-webkit-transition:all 0.5s;-moz-transition:all 0.5s}.family-tree li{float:left;text-align:center;list-style-type:none;position:relative;padding:20px 5px 0 5px}.family-tree>ul>li:first-child::before, .family-tree>ul>li:first-child::after{border-top:none;height:10px;margin-top:10px}.family-tree li::before, .family-tree li::after{content:"";position:absolute;top:0;right:50%;border-top:2px solid rgb(204, 204, 204);width:50%;height:20px}.family-tree li::after{right:auto;left:50%;border-left:2px solid rgb(204, 204, 204)}.family-tree li a{border:2px solid rgb(204, 204, 204);padding:5px;text-decoration:none;color:rgb(0, 0, 0);font-family:Arial, sans-serif;display:inline-block;border-radius:5px;background:rgb(255, 255, 255);position:relative; min-width:300px}.family-tree li a img{width:100px;height:100px;border-radius:50%, align:center}.family-tree li a .name{display:block;margin-top:5px}.family-tree>ul>li:first-child>a::after{content:"A founding father of Computer Science as a separate discipline. Was the first recipient of the Turing Award"}.family-tree li:first-child>a::after{position:absolute;top:-55px;left:50%;transform:translateX(-50%);white-space:normal;width:250px;padding:5px 10px;border-radius:10px;background-color:rgb(240, 240, 240);border:1px solid rgb(204, 204, 204);font-size:12px;text-align:center}.family-tree>ul>li>a{font-size:16px;font-weight:bold}.family-tree ul ul::before{content:"";position:absolute;top:0;left:50%;border-left:2px solid rgb(204, 204, 204);width:0;height:20px}.family-tree li a:hover, .family-tree li a:hover+ul li a{border:2px solid rgb(148, 160, 180)}</style><div class="family-tree"> <ul> <li> <a target="_blank" href="https://en.wikipedia.org/wiki/Alan_Perlis" title="Alan Perlis (1922–1990)"><!--img src="https://upload.wikimedia.org/wikipedia/en/5/59/Alan_Perlis.jpg" alt="Alan Perlis (1922–1990)"--><span class="name">Alan Perlis (1922–1990) <br/> (Purdue, CMU, Cal Tech, Yale)</span></a> <ul> <li> <a target="_blank" href="https://c21u.gatech.edu/directory/person/spencer-rugaber" title="Yale Spencer Rugaber"><!--img src="Spencer_Rugaber.webp" alt="1978 Yale Spencer Rugaber  (Georgia Tech.)"--><span class="name">1978 Yale Spencer Rugaber  (Georgia Tech.)</span></a> <ul> <li> <a target="_blank" href="https://www.chrisparnin.me" title="2014 Chris Parnin (North Carolina State U.)"><!--img src="/media/chris_parnin.jpeg" alt="2014 Chris Parnin (North Carolina State U.)"--><span class="name">2014 Chris Parnin (North Carolina State U.)</span></a> <ul> <li> <a target="_blank" href="https://chbrown13.github.io/" title="2021 Chris Brown (Virginia Tech)"><!--img src="/media/chris_brown.jpeg" alt="2021 Chris Brown (Virginia Tech)"><span class="name"-->2021 Chris Brown (Virginia Tech)</span></a> <ul> <li > <a target="_blank" href="#" title="Me"><!--img src="/media/profile.JPG" alt=""--><span class="name">I am Here</span></a> </li> </ul> </li> </ul> </li> </ul> </li> </ul> </li> </ul></div><br/> <h4 style="clear:both; padding-top:10px">&#169; 2024 Advisee tree credit <a target="_blank" href="https://www.chrisparnin.me">Chris Parnin </a> </h4>

    design:
      columns: '2'
      css_style: ''
---
