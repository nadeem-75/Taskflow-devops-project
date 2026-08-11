# Use Node.js as the base image
FROM node:20-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy application code
COPY . .

# Application listens on port 3000
EXPOSE 3000

# Start the Node.js application
CMD ["npm", "start"] 
