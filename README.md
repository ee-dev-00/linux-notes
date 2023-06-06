# Basic

### Find
Search directories by name

    find ./external -type d -name '.git*'

Search with exec

    find ./external -type d -name '.git*' -exec rm -rf {} \;



