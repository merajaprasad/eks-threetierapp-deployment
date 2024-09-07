# Stage 1: Build
FROM node:14 as build

# Set the working directory in the container
WORKDIR /usr/src/app

# Copy package.json and package-lock.json for dependency installation
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application files
COPY . .

# Stage 2: Final Image
FROM node:14-slim

# Set the working directory in the final image
WORKDIR /usr/src/app

# Copy only the necessary files from the builder stage
COPY --from=build /usr/src/app .

# Run the application
CMD [ "npm", "start" ]

